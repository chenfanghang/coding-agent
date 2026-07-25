# Pico 源码级面试追问与技术内幕

本文深入 `pico` 源代码底层实现，收录核心模块的代码片段、数据结构与并发控制逻辑，并针对源码追问提供防守提示。

---

## 1. 核心源码结构与类设计

```text
pico/
├── core/
│   ├── runtime.py               # Pico / Runtime 门面类，控制生命周期与密匙脱敏
│   ├── engine.py                # AgentEngine 主循环调度器 (ReAct Engine)
│   ├── context_manager.py       # ContextManager Prompt 组装与预算计算
│   ├── compact.py               # CompactManager 历史对话压缩器
│   ├── turn_history.py          # TurnHistoryBuilder Turn 归组与 Transcript 渲染
│   ├── tool_executor.py         # ToolExecutor 工具执行网关、参数校验与元数据收集
│   ├── tool_profiles.py         # ToolProfile 动态工具面过滤 (default/plan/readonly/dream)
│   ├── permissions.py           # PermissionChecker 风险审批策略与安全拦截
│   ├── plan_mode.py             # PlanModeController Plan 模式控制与 implementation_plan 生成
│   ├── todo_ledger.py           # TodoLedger Task 拆解与待办清单流转
│   ├── worker_manager.py        # WorkerManager Worker 子 Agent 派发与沙箱隔离
│   ├── runtime_checkpoints.py   # Checkpoint 创建、恢复评估与身份解绑
│   ├── session_store.py         # SessionStore 读写 .pico/sessions/*.json
│   └── run_store.py            # RunStore 写入 .pico/runs/<run_id>/ (trace/report)
└── features/
    ├── memory.py                # LayeredMemory, DurableMemoryStore, Auto-Dream
    └── sandbox/                 # Shell 命令安全隔离防护沙箱
```

---

## 2. 核心代码块 1：`_build_prompt_and_metadata` 6 步流

在 [pico/core/runtime.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/runtime.py) 中：

```python
def _build_prompt_and_metadata(self, user_message):
    # Step 1: 刷新工作区状态与 System Prefix 指纹
    refresh = self.refresh_prefix()
    # Step 2: 评估 Checkpoint 过期与恢复状态
    self.resume_state = self.evaluate_resume_state()
    # Step 3: 初次尝试组装 Prompt
    prompt, metadata = self.context_manager.build(user_message)
    
    # Step 4: 检查 Token 超预算自适应压缩
    if (
        metadata.get("prompt_over_budget")
        and len(self.session.get("history", [])) > 4
    ):
        self.compact_history(trigger="auto_prompt_over_budget")
        prompt, metadata = self.context_manager.build(user_message)
        metadata["auto_compacted"] = True

    # Step 5: 丰富 Trace 与 Prompt Cache 元数据
    metadata.update({
        "prefix_chars": len(self.prefix),
        "workspace_chars": len(self.workspace.text()),
        "memory_chars": len(self.memory_text()),
        "history_chars": len(self.history_text()),
        "request_chars": len(user_message),
        "prefix_hash": self.prefix_state.hash,
        "prompt_cache_key": self.prefix_state.hash,
        "workspace_fingerprint": self.prefix_state.workspace_fingerprint,
        "prompt_cache_supported": bool(getattr(self.model_client, "supports_prompt_cache", False)),
        "resume_status": self.resume_state.get("status", CHECKPOINT_NONE_STATUS),
        "stale_paths": list(self.resume_state.get("stale_paths", [])),
    })
    metadata.update(self.detected_secret_env_summary())
    
    # Step 6: 广播事件
    self.session_event_bus.emit("context_usage_recorded", {
        "run_id": getattr(getattr(self, "current_task_state", None), "run_id", ""),
        "context_usage": metadata.get("context_usage", {}),
    })
    return prompt, metadata
```
🛡️ **【源码考点防守】**：这里在组装后再判断 `prompt_over_budget` 并重新构建，避免了预估误差导致的模型溢出错误。

---

## 3. 核心代码块 2：`CompactManager` 压缩算法与结构

在 [pico/core/compact.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/compact.py) 中：

```python
def compact(self, trigger="manual", keep_recent_turns=2):
    history = list(self.agent.session.get("history", []))
    groups = self._group(history)
    if len(groups) <= keep_recent_turns:
        return self._summary(trigger, history, history, "")

    compacted_turns = groups[:-keep_recent_turns]
    kept_turns = groups[-keep_recent_turns:]
    compacted_items = [item for _, items in compacted_turns for item in items]
    kept_items = [item for _, items in kept_turns for item in items]
    
    summary_text = self._summary_text(compacted_items)
    summary_item = self.agent.turn_history.enrich({
        "role": "system",
        "kind": "compact_summary",
        "content": summary_text,
        "created_at": now(),
        "source": "compact",
    })
    self.agent.session["history"] = [summary_item, *kept_items]
    summary = self._summary(trigger, history, self.agent.session["history"], summary_text)
    self.agent.session.setdefault("compactions", []).append(summary)
    self.agent.session_path = self.agent.session_store.save(self.agent.session)
    return summary
```
🛡️ **【源码考点防守】**：`groups[:-keep_recent_turns]` 保证了按 Turn 为最小裁剪粒度，不会出现切掉半个工具调用参数的语义断层。

---

## 4. 核心代码块 3：`retrieval_candidates` 检索打分源码

在 [pico/features/memory.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory.py#L711-L726) 中：

```python
def retrieval_candidates(self, query, limit=3):
    query_tokens = _tokenize(query)
    ranked = []
    for topic in self.load_index():
        notes = self.load_topic_notes(topic["topic"])
        for note in notes:
            note_tags = {tag.lower() for tag in note.get("tags", [])}
            note_tokens = _tokenize(note.get("text", "")) | _tokenize(topic.get("title", "")) | note_tags
            
            exact_tag_match = int(bool(query_tokens & note_tags))
            keyword_overlap = len(query_tokens & note_tokens)
            
            if exact_tag_match == 0 and keyword_overlap == 0:
                continue
                
            recency = _parse_timestamp(note.get("created_at"))
            ranked.append(((exact_tag_match, keyword_overlap, recency), note))
            
    # 按照 (exact_tag_match, keyword_overlap, recency) 元组降序排序
    ranked.sort(key=lambda item: item[0], reverse=True)
    return [note for _, note in ranked[:limit]]
```
🛡️ **【源码考点防守】**：采用元组排序 `(exact_tag_match, keyword_overlap, recency)`，利用 Python 默认的元组对比特性直接实现了三级优先级降序。

---

## 5. 核心代码块 4：Worker Sub-Agent 派发与 `write_scope` 隔离

在 [pico/core/worker_manager.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/worker_manager.py) 中：

```python
class WorkerManager:
    def __init__(self, agent):
        self.agent = agent
        self._workers = {}

    def spawn_worker(self, name, task_prompt, write_scope=None):
        from .runtime import Pico
        worker_runtime = Pico(
            model_client=self.agent.model_client,
            workspace=self.agent.workspace,
            session_store=self.agent.session_store,
            approval_policy="auto",
            write_scope=write_scope,  # 硬性锁定写权限目录
            auto_dream=False,
        )
        thread = threading.Thread(
            target=self._run_worker,
            args=(name, worker_runtime, task_prompt),
            daemon=True
        )
        thread.start()
```
🛡️ **【源码考点防守】**：`write_scope` 参数传入后，在 Worker 内部的 ToolExecutor 中硬拦截，阻止写权限越界。

---

## 6. 核心代码块 5：Tool Profile 动态工具集过滤

在 [pico/core/tool_profiles.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_profiles.py) 中：

```python
TOOL_PROFILES = {
    "default": ALL_TOOLS,
    "plan": ["read_file", "dir_list", "grep_search", "write_plan"],  # 物理遮蔽写代码与 Shell
    "readonly": ["read_file", "dir_list", "grep_search"],
    "dream": ["read_file", "write_file", "patch_file"],
}

def filter_tools_for_profile(profile_name, tools):
    allowed_names = set(TOOL_PROFILES.get(profile_name, ALL_TOOLS))
    return [tool for tool in tools if tool.name in allowed_names]
```

---

## 7. 核心代码块 6：并发互斥文件锁实现

在 [pico/features/memory.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory.py) 中：

```python
def acquire_lock(memory_dir):
    ensure_memory_dir(memory_dir)
    lock_path = _lock_path(memory_dir)
    current_pid = os.getpid()
    try:
        stat = lock_path.stat()
        age = datetime.now().timestamp() - stat.st_mtime
        holder_pid = int(lock_path.read_text(encoding="utf-8").strip())
        if age < HOLDER_STALE_S:
            try:
                os.kill(holder_pid, 0)
                return False  # 锁被有效持有着，返回 False
            except OSError:
                pass
    except (OSError, ValueError):
        pass
    lock_path.write_text(str(current_pid), encoding="utf-8")
    return True
```
🛡️ **【源码考点防守】**：利用 `os.kill(pid, 0)` 发送信号检查持锁进程是否仍然存活，兼顾了文件锁的抢占与 Stale 进程锁自动释放。
