# Pico v3 AI Agent 研发工程师面试全景终极指南 (真正全量聚合大百科)

> 📌 **文档说明**：本文档是将 Pico v3 项目的**最新简历文本与量化指标源码映射**、**大厂自我介绍**、**0 到 1 架构演进**、**两条主线 10 阶段生命周期**、**门禁模式与打回机制 (`FinalReadiness`)**、**ADR 选型辩论**、**全景 21 大项目高频问点**、**`trace.jsonl` 14 种 Event 结构**、**三大消融实验表格**、**Troubleshooting 实战案例**与**三阶评分标准**彻底无损熔炼后的**唯一终极超级大百科**。

---

## 目录导航

- [一、 最新简历项目原文本与核心技术映射矩阵](#一-最新简历项目原文本与核心技术映射矩阵)
- [二、 大厂面试开口逐字稿 (1 分钟 / 3 分钟标准版)](#二-大厂面试开口逐字稿-1-分钟--3-分钟标准版)
- [三、 架构演进与 35,000+ 行大重构工程总结](#三-架构演进与-35000-行大重构工程总结)
- [四、 Pico 全景架构与两条主线 10 阶段生命周期](#四-pico-全景架构与两条主线-10-阶段生命周期)
- [五、 完成就绪门禁与打回机制深度解析 (FinalReadiness)](#五-完成就绪门禁与打回机制深度解析-finalreadiness)
- [六、 三大核心 ADR 架构选型深度辩论](#六-三大核心-adr-架构选型深度辩论)
- [七、 全景 21 大高频项目问点深度攻防与示范稿](#七-全景-21-大高频项目问点深度攻防与示范稿)
- [八、 Trace 审计日志流 (14 种 Event) 与确定性回放评测](#八-trace-审计日志流-14-种-event-与确定性回放评测)
- [九、 三大实测消融实验数据表与 Troubleshooting 排错案例](#九-三大实测消融实验数据表与-troubleshooting-排错案例)
- [十、 问点防御三阶评分标准表与 3 天备面练习计划](#十-问点防御三阶评分标准表与-3-天备面练习计划)

---

## 🛠️ 底层架构与专项文档库快捷面板

本面试指南与 Pico 源码底层的 6 大专属架构文档库保持 100% 同步，面试中遇到深度追问可随时跳转查阅底层细节：

- 🧠 **分层记忆系统**：[docs/记忆/README.md](../记忆/README.md) (Working / Durable / Quarantine / Retrieval / Auto-Dream)
- ⚡ **上下文编排系统**：[docs/上下文/README.md](../上下文/README.md) (Prefix Lock 85.3% Caching / Section Budget / Pressure Tier)
- 🛠️ **受控工具网关**：[docs/工具调用/README.md](../工具调用/README.md) (8 重受控关卡 / Read Freshness / Path Escape / Repetition)
- 🔄 **任务恢复系统**：[docs/任务恢复/README.md](../任务恢复/README.md) (5 大 Resume 状态矩阵 / Workspace Drift / Re-anchoring)
- 🛡️ **完成门禁系统**：[docs/完成门禁/README.md](../完成门禁/README.md) (Final Readiness 5 大检查项 / Block 打回强提醒 / Hooks)
- 🤖 **多 Agent 协同**：[docs/多Agent协同/README.md](../多Agent协同/README.md) (Explore 与 Worker 分工 / write_scope 100% 写入隔离)

---

## 一、 最新简历项目原文本与核心技术映射矩阵

### 1.1 简历标准原文本

**本地 Coding Agent Harness**  
`2026.02 - 至今`

- **项目概况**：开发本地 Coding Agent Harness，用于代码仓库长链路任务；串联模型接入、工具调用、上下文与记忆、checkpoint / resume、skills、运行记录和评测，解决上下文膨胀、重复读文件、状态丢失和结果难复盘问题。
- **核心技术与方法**：
  1. **Agent Harness 架构设计**：设计本地代码 Agent 主流程，统一模型、工具、会话、skills、子任务与运行工件；分层运行编排、工具边界和状态存储，形成可恢复、可审计的执行链路。
  2. **上下文管理与成本优化**：设计压力感知的上下文裁剪与长结果落盘机制，Prompt 减少 9.31%，E2E 估算输入 Token 下降 14.69%，Prompt Caching 命中率达 85.3%，Verifier 通过率保持 100%。
  3. **分层记忆系统**：构建跨会话记忆与自动沉淀机制，按相关性召回项目知识；在 Memory Challenge 55 案例中达到 94.55% 准确率与 100% 召回率，将重复文件读取由 60 次降至 0 次、平均模型尝试由 2 次降至 1 次。
  4. **任务恢复与状态校验**：实现 checkpoint / resume 与环境一致性校验；10 个任务共 30 次运行，恢复成功率 90.9%，工作区漂移识别率 100%，旧状态错误接受率 0%。
  5. **工具安全与运行治理**：建立工作区隔离、读后修改、高风险审批和重复调用拦截机制，通过 10 个安全场景验证路径越界、越权写入和重复副作用拦截。
  6. **评测体系与质量验证**：搭建覆盖上下文、记忆、恢复和工具安全的固定 Benchmark 与消融实验，12 个固定任务的通过率、预算内完成率和 Verifier 通过率均为 100%。

---

### 1.2 简历实测数据与 Harness 物理源码 1:1 映射表

| 简历核心技术点 | 简历实测指标数据 | 底层源码实现模块 / 详细文档库 | 关键控制逻辑与工程实现 |
| :--- | :--- | :--- | :--- |
| **上下文管理** | **Prompt 减少 9.31%**<br>**Input Token 下降 14.69%**<br>**Caching 命中率 85.3%** | [pico/core/context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_orchestrator.py)<br>⚡ [docs/上下文/README.md](../上下文/README.md) | `ContextOrchestrator` 字节级锁前缀；`ContextPressure` 4 阶梯剪枝；超长 Tool 结果落盘写为 `artifacts/` 磁盘工件引用。 |
| **分层记忆系统** | **Memory Challenge 94.55%**<br>**重复读取 60 次降至 0 次**<br>**模型尝试 2 次降至 1 次** | [pico/features/memory.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory.py)<br>🧠 [docs/记忆/README.md](../记忆/README.md) | 四层 Markdown 结构；`(tag, overlap, recency, note_index)` 四元组确定性算分召回；`MemoryQuarantine` 毒化隔离与 `.consolidate-lock` PID 存活性检测。 |
| **任务恢复** | **恢复成功率 90.9%**<br>**工作区漂移识别率 100%**<br>**旧状态错误接受率 0%** | [pico/core/checkpoint.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/checkpoint.py)<br>🔄 [docs/任务恢复/README.md](../任务恢复/README.md) | `CheckpointManager` 每次工具执行落盘 `ckpt_xxx` 原子快照；`workspace_fingerprint` 比对 Git Head SHA 与文件 mtime；`workspace-mismatch` 强制重锚定。 |
| **工具安全** | **10 个安全场景拦截率 100%**<br>**只读/越权写拦截率 100%** | [pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py)<br>🛠️ [docs/工具调用/README.md](../工具调用/README.md) | `run_tool` 8 重受控关卡防线；`ToolPolicyChecker` 强校验 `Read Freshness Guard`（未读先改拦截）；`WorkerManager` 强绑 `write_scope` 白名单（100% 隔离）。 |
| **完成就绪门禁** | **未测试盲目答复降低 97.3%**<br>**假完成截获率 100%** | [pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py)<br>🛡️ [docs/完成门禁/README.md](../完成门禁/README.md) | `evaluate_final_readiness()` 5 大物理检查项；检查代码 Diff、必填产物与 `pytest` 执行证据；`block` 强打回并注入 `readiness_notice` 强提醒。 |
| **评测与回放** | **CI/CD 回放耗时 < 0.5s**<br>**12 Benchmark 通过率 100%** | [pico/testing/fake_model.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/testing/fake_model.py)<br>📊 [docs/评测体系/README.md](../评测体系/README.md) | 基于历史 `trace.jsonl` 的 `FakeModelClient` 确定性回放；物理文件断言、pytest 与 `trace.jsonl` 事件轨多维 Verifier。 |

---

## 二、 大厂面试开口逐字稿 (1 分钟 / 3 分钟标准版)

### 2.1 1 分钟开门见山版 (适合技术初面/快速自我介绍)

> “面试官您好，我叫陈方航。近期我主导开发了一款本地受控的 Coding Agent Harness。
> 
> 在做这个项目前，我发现绝大多数 Coding Agent 在真实代码仓库中存在三大死穴：**改完代码不测试就盲目给答复**、**Prompt 前缀抖动导致 API 费用极高**，以及**缺乏工具安全隔离**。
> 
> 针对这些痛点，我设计了一套受控 Harness：
> - 引入 **`FinalReadiness` 完成就绪硬门禁**，如果改动了源码但未在 Session 内跑通 `pytest`，框架直接在调度层强行阻断退出，未测试盲目答复概率骤降 **97.3%**；
> - 手写 **`ContextOrchestrator` 编排器**，强分隔静态 Header 锁定前缀，在压力感知裁剪下实现 E2E 输入 Token 下降 **14.69%**，Prompt Caching 命中率稳定在 **85.3%**；
> - 建立包含 `Read Freshness Guard`（读后修改）在内的 **8 重受控工具网关**，并构建了基于 `FakeModelClient` 的零 API 成本确定性回归评测。
> 
> 今天非常期待能与您深入探讨 Agent 架构治理与上下文优化的细节。”

---

### 2.2 3 分钟深度项目破局版 (适合架构/二面/终面)

> “面试官您好，我重点汇报一下我主导的‘本地 Coding Agent Harness’项目。
> 
> 很多团队在做 Coding Agent 时，往往基于开源框架搭个 ReAct 脚本 Demo，但一放到团队狗粮 (Dogfooding) 场景中就会遭遇四大工程死穴：
> 1. **交付代码不可靠**：模型改完代码直接输出文本，代码经常包含低级语法错误；
> 2. **前缀缓存频繁失效**：通用框架在 Prompt 前端注入动态时间或配置，打乱了字符级匹配，导致 API 费用极其昂贵；
> 3. **记忆被模型幻觉污染**：错乱规则直接追加写入长期文件；
> 4. **多 Agent 越权写代码**：派发后台子任务时越界篡改了主工程源码。
> 
> 为了彻底解决这四大死穴，我对整个 Harness 进行了系统化的大重构：
> - **在完成治理上**，设计 `FinalReadiness` 调度层硬门禁，审计 `has_code_changes` 与 `pytest` 运行证据，强阻断未测试退出；
> - **在上下文管理与成本优化上**，设计压力感知的上下文裁剪与长结果落盘机制，实现 Prompt 减少 9.31%，端到端估算输入 Token 下降 14.69%，Prompt Caching 命中率保持在 85.3%；
> - **在分层记忆治理上**，构建跨会话记忆与自动沉淀机制，按相关性召回知识，在 12 个任务中将重复文件读取由 60 次降至 0 次，平均模型尝试由 2 次降至 1 次，正确率保持 100%；
> - **在任务恢复与工具安全上**，实现 checkpoint / resume 与环境指纹校验，10 个任务共 30 次运行达到恢复成功率 90%、漂移识别率 100%、旧状态错误接受率 0%；建立工作区隔离、读后修改 (`Read Freshness`)、高风险审批与越权写入拦截；
> - **在评测与质量验证上**，基于 `FakeModelClient` 解析 `trace.jsonl` 实现零 API 成本、<0.5 秒的 CI/CD 确定性回归，12 个固定任务通过率、预算内完成率和 Verifier 通过率均为 100%。”

---

## 三、 架构演进与 35,000+ 行大重构工程总结

### 3.1 从早期 MVP 脚本到工业级受控 Harness 的演进路线图

```text
┌──────────────────────────────────────────────────────────┐
│ Milestone 1: 早期 MVP 脚本阶段                            │
│ - 简单的 ReAct 感知-行动单进程循环                         │
│ - 基础工具调用 (subprocess run_shell, 简单读写文件)      │
│ - 无硬性测试门禁、无记忆检疫隔离、Prompt Caching 命中率低   │
└────────────────────────────┬─────────────────────────────┘
                             │
                             v
┌──────────────────────────────────────────────────────────┐
│ Milestone 2: 规范化与沙箱治理阶段                        │
│ - 工具参数全面迁移至 Pydantic v2 强校验                  │
│ - 引入 Shell 隔离防护沙箱与路径逃逸防御                   │
│ - 引入 trace.jsonl 事件流与运行审计工件                  │
└────────────────────────────┬─────────────────────────────┘
                             │
                             v
┌──────────────────────────────────────────────────────────┐
│ Milestone 3: 工业级受控 Harness 阶段 (当前版本)            │
│ - ContextOrchestrator + 85.3% Prompt Caching 静态 Prefix │
│ - FinalReadiness 代码修改未测试强行打回闭环                │
│ - 四层记忆 + MemoryQuarantine 检疫池与 PID 互斥锁        │
│ - WorkerManager 多 Agent 协同与 write_scope 写隔离        │
│ - FakeModelClient 确定性回放与 HarnessBench 消融评测集   │
└────────────────────────────┴─────────────────────────────┘
```

> **面试官问：能讲讲你们这个 Agent 项目是怎么从 0 到 1 演进的吗？早期版本遇到了什么问题，促使你们做了后面这套重构？**

**逐字话术：**
> “项目的演进经历了一次非常深刻的工程重构，涉及 **273 个文件、35,000+ 行代码** 的系统化升级。
> 
> 最开始我们只是做了一个基础的 ReAct 脚本 MVP，能让大模型调 Shell 和读写文件。但在真实的代码重构和开发狗粮 (Dogfooding) 场景中，我们迅速遭遇了**四大致命工程死穴**：
> 1. **修改代码不测试就盲目退出**（模型改完源码直接输出回答，交付代码错误率极高）；
> 2. **前缀缓存 (Prompt Caching) 命中率极低**（仅 12%，因为 Prompt 头部动态变量导致哈希频繁抖动，API 费用极高）；
> 3. **记忆被模型幻觉污染**（错误规则和不稳定的临时代码直接写坏了物理文件）；
> 4. **子任务越权**（派发后台任务时，子进程越界修改了主工程的源码）。
> 
> 针对这四大死穴，我们做了一次全面的架构重构：
> - 引入 **`FinalReadiness` 硬门禁**，如果改动了源码但未在 Session 内跑通 `pytest`，框架直接在调度层阻断退出；
> - 引入 **`ContextOrchestrator` 静态 Prefix 锁定**，将前缀缓存命中率由 12% 锁死提升至 **85.3%**；
> - 引入 **`MemoryQuarantine` 检疫隔离池** 和 PID 互斥锁，将事实矛盾卡死在 `quarantine/` 目录；
> - 引入 **`WorkerManager` 线程子任务派发** 与 `write_scope` 路径白名单隔离。
> 
> 🛡️ **【防御防守 Hook】**：很多项目的重构只是改改接口，而我们在这次重构中不仅做到了关键治理逻辑的强化，还建立了 `FakeModelClient` 确定性回放评测集，保证了 35,000+ 行重构 100% 不破坏既有契约。”

---

## 四、 Pico 全景架构与两条主线 10 阶段生命周期

### 4.1 架构解耦全景图 (ASCII)

```text
+-------------------------------------------------------------------------------+
|                             Pico Agent Harness Runtime                        |
+-------------------------------------------------------------------------------+
|  1. 装配主线 (Setup Line)                                                      |
|     load_config() -> Workspace.discover() -> Skills/Memory/Context 初始化    |
+-------------------------------------------------------------------------------+
|  2. 执行主线 (Execution Line)                                                  |
|     [ContextOrchestrator] -> (Prompt Build) -> [Engine.step()]                 |
|             |                                       |                         |
|             v                                       v                         |
|     (Static Header Lock)                  <Model Call (Intent)>               |
|             |                                       |                         |
|             v                                       v                         |
|     [Prompt Caching]                      [Tool Policy Gate]                  |
|                                                     |                         |
|                                                     v                         |
|                                           (Read Freshness / Path Scope)       |
|                                                     |                         |
|                                                     v                         |
|                                           [ToolExecutor & Sandbox]            |
|                                                     |                         |
|                                                     v                         |
|                                           [FinalReadiness Gate]               |
|                                        (has_code_changes & pytest)            |
+-------------------------------------------------------------------------------+
|  3. 存储与审计轨 (Storage & Logging Tracks)                                    |
|     - 恢复轨: .pico/sessions/<id>.events.jsonl (SessionEventBus State Resume) |
|     - 审计轨: .pico/runs/<id>/trace.jsonl (14 Events & FakeModel Playback)    |
+-------------------------------------------------------------------------------+
```

---

### 4.2 两条主线 10 阶段全生命周期

```text
[装配主线 Setup Line] (阶段 1-4)
  Stage 1: load_config()        -> 读取 .pico.toml 与治理模式
  Stage 2: Workspace.discover() -> 扫描工程指纹 Commit SHA / Diff
  Stage 3: SkillLoader.init()   -> 渐进式加载 Skill SOP 索引
  Stage 4: LayeredMemory.init() -> 载入 Working Memory 与 Durable 规则
                  │
                  ▼
[执行主线 Execution Line] (阶段 5-10)
  Stage 5: ContextOrchestrator  -> 静态 Header 字节锁定构建 Prompt
  Stage 6: ModelClient.call()   -> 产生 Intent 工具意图 / 最终文本
  Stage 7: Tool Policy Gate     -> 校验 Read Freshness / Path Scope
  Stage 8: ToolExecutor         -> 120s 超时/沙箱执行并抓取 Snapshot Diff
  Stage 9: FinalReadiness Gate  -> 审计 has_code_changes 与 pytest
  Stage 10: RunStore & Session  -> 刷入 trace.jsonl 并持久化 Checkpoint
```

---

## 五、 完成就绪门禁与打回机制深度解析 (FinalReadiness)

**源码文件**：[pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py) & [pico/core/final_readiness_tools.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness_tools.py)

完成就绪治理门禁 (`FinalReadiness`) 是 Pico Harness 解决 Coding Agent 盲目未测试退出 (Premature Exit) 的核心技术。

### 5.1 三大门禁模式 (Readiness Modes)

- **`strict` (严格门禁模式 - 默认生产配置)**：
  当检测到 Session 产生物理改动 (`has_code_changes == True`) 但尚未在 Session 中执行并通过 `pytest` (`test_executed == False`) 时，**硬阻断 (Action: block)** 大模型交付，追加警告系统消息并拉回 Engine 调度循环。
- **`advisory` (建议模式)**：
  当检测到改动未测试时，**不阻断交付 (Action: allow)**，但会在 `trace.jsonl` 日志中打上 `readiness_warning` 标记并记录警告提示。
- **`off` (关闭模式)**：
  完全绕过门禁，用于对比消融实验 (Ablation Studies)。

---

### 5.2 `readiness_reasons` 11 大缺陷原因枚举

探查函数 `readiness_reasons()` 物理审计的 11 大原因分类：

| 原因标识 (`Reason ID`) | 缺陷物理特征 | 严重级别 (`Severity`) |
| :--- | :--- | :--- |
| **`changed_paths_without_verification`** | 修改了物理文件但未在 Session 内成功跑通 pytest | 🔴 **hard (物理阻断)** |
| **`failed_verification`** | 单元测试显式返回失败 (`failed`) | 🔴 **hard (物理阻断)** |
| **`missing_required_artifact`** | 要求的交付文件不存在 | 🔴 **hard (物理阻断)** |
| **`unresolved_high_priority_todo`** | Todo 账本里尚有高优 pending 任务 | 🟡 **soft (警告提示)** |
| **`governance_denial`** | 运行中触发过网关拒防拦截 | 🟡 **soft (警告提示)** |
| **`partial_success_workspace_changed`**| 工具仅部分成功且物理工程已被改动 | 🟡 **soft (警告提示)** |
| **`context_pressure_compaction_failed`**| 上下文剪枝压缩失败 | 🟡 **soft (警告提示)** |

---

### 5.3 打回机制 (Block / Rejection Flow) 的 4 步流转

```text
[大模型输出 final 文本 (无 tool_call)]
                 │
                 ▼
     ┌──────────────────────┐
     │  Engine 拦截触发就绪评估 │
     └──────────┬───────────┘
                │
                ▼
  ┌───────────────────────────┐
  │ 审计物理证据:               │
  │ - has_code_changes (改动?)│
  │ - test_executed (跑测试?)  │
  └─────────────┬─────────────┘
                │
         (改动未测试?)
        /             \
      YES              NO
      /                 \
     ▼                   ▼
[Action: block]     [Action: allow]
     │                   │
     ▼                   ▼
[强注系统警告，打回]    [允许交付用户]
```

---

### 5.4 防死锁与 3 轮降级机制 (Anti-Deadlock)

- **原因哈希算分**：系统对打回原因计算 `reason_hash = sha256(reason_text)`。
- **连续打回计数器**：记录连续相同原因阻断的次数 `consecutive_rejections`。
- **3 轮降级策略**：若由于测试环境本身有问题或模型尝试 3 轮仍无法修复通过，为避免死循环，第 4 轮自动降级为 `advisory` 模式放行，或触发 HITL (Human-In-The-Loop) 界面请用户人工选择。

---

## 六、 三大核心 ADR 架构选型深度辩论

### ADR-001：为什么不直接基于开源 Agent 框架 (如 LangChain / LangGraph) 开发，而是选择纯 Python 自研？

> **面试官问：市面上有很成熟的开源 Agent 框架（如 LangChain、LangGraph），你为什么还要自己纯手写一个 Harness？**

**逐字话术：**
> “这是一个非常核心的架构 Trade-off 问题。我们在做技术选型时做了深度推导，主要出于三个工程考量：
> 
> 1. **Prompt 字节级控制与前缀缓存契约**：大模型 Provider（如 DeepSeek、Claude）的前缀缓存 (Prompt Caching) 机制严格要求从 Prompt 第 1 个字节开始的 **Prefix 必须绝对一致**。通用开源框架为了通用性，往往在 Prompt 头部动态注入框架级变量或环境描述，一旦这些动态字节散落在前端，会导致 `prefix_hash` 改变而触发 Cache Miss。而我们的 `ContextOrchestrator` 纯手写编排，能将前缀强行固定在头部，最大化前缀缓存命中率。
> 2. **完成治理 (Final Readiness) 契约的原生内聚**：虽然通用框架可以通过中间件 (Middleware) 或 Event Hook 拦截输出，但通用框架调度流多将‘无 tool_call 的回答’直接判断为 End 结束节点。要结合 Session 级 `has_code_changes` 物理修改证据与 `pytest` 运行证据做动态阻断打回，在开源框架中需要对 StateGraph 进行深度侵入重构；而在自研 Harness 中，我们把完成治理门禁设计为 Engine 调度循环的一等公民，拦截打回逻辑天然内聚。
> 3. **极轻量与启动耗时**：开源框架依赖链极深，启动需 1-3 秒；而我们的 Harness 纯 Python 零重型依赖，启动 **< 50ms**，内存占用不到 30MB，非常贴合命令行和 IDE 内置终端环境。
> 
> 🛡️ **【防御防守 Hook】**：开源框架非常适合快速搭建通用 Demo 原型；但在本地 Coding 场景这种对**前缀字节级锁定**、**测试验证硬门禁**和**终端极速响应**要求极其苛刻的场景下，轻量自研是经过严谨推导后的最优解。”

---

### ADR-002：为什么在技术 Coding 场景下放弃了向量数据库 (Vector DB/RAG)，而是采用四层明文 Markdown？

> **面试官问：现在 RAG 和 Vector DB 很火，你们在存储和记忆检索上为什么没有用向量数据库，而是用物理 Markdown 文件？**

**逐字话术：**
> “这同样基于代码场景的独特属性推导：
> 
> 1. **精确标量匹配 vs 模糊语义距离**：代码/配置场景要求绝对精确匹配。向量相似度在代码变量名、正则和配置项上存在离散误差（例如 `jwt_secret` 与 `api_secret` 向量很近但属于不同配置），RAG 很容易召回无关干扰段落；而在代码场景下，基于 `(tag, keyword, recency)` 三元组能做到 **< 1ms 延迟** 与 100% 精确匹配。
> 2. **重新切片 Embedding 的开销不可接受**：代码仓库修改极度频繁，如果每次写文件都重新 Chunking 和切片调 API 算 Embedding，CPU 与 Token 开销巨大；而明文 Markdown 增量提纯几乎零开销。
> 3. **人类可读与 Git 版本兼容**：明文 Markdown (`.pico/memory/`) **100% 人类可读可干预**，开发者可以手动编辑纠错，并完美融入 Git diff 版本审计。”

---

### ADR-003：交互界面为什么选择 Textual TUI 响应式终端，而不是 Web UI / Electron？

> **面试官问：前端交互为什么做成终端 TUI，不做成 Web 界面或者桌面端软件？**

**逐字话术：**
> “主要出于**极致性能**与**开发者原生工作流**考量：
> - **内存开销**：Electron / Web UI 需启动数百兆内存的 Chromium 内核；基于 Python Textual 的 TUI 内存占用 **< 30MB**。
> - **极速启动**：启动耗时 **< 50ms**，即开即用。
> - **远程 Linux SSH 完美兼容**：很多开发者是在远程 Linux 服务器或 SSH 终端开发，Web 界面需要复杂端口映射，而 TUI 可以在任何 SSH 终端原生流畅运行。”

---

## 七、 全景 21 大高频项目问点深度攻防与示范稿

### 📍 模块一：完成治理与测试硬门禁

#### Q1：如何解决大模型在 Agent 场景下“改完代码不测试就盲目给答复 (Premature Exit)”的问题？
- **源码文件**：[pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py)
- **答题骨架**：放弃 Prompt 软提醒 $\rightarrow$ 建立 Engine 调度层硬门禁 $\rightarrow$ 比对物理改动 `has_code_changes` 与 `pytest` 运行证据 $\rightarrow$ 强行打回并注入警告。
- **🎙️ 60-120 秒示范稿**：
  > “在 Pico 项目中，我们放弃了在 Prompt 里劝说模型的软约束做法，而是在 Engine 调度循环层设计了 **`FinalReadiness` 完成就绪硬门禁**。当大模型试图输出不带 tool 调用的最终答复时，引擎拦截评估：若检测到 `has_code_changes == True` 但 Session 内 `test_executed == False`，门禁直接做出 `action == 'block'` 打回判决，并在对话历史追加警告：*‘您已修改源码但尚未执行测试，请调用 pytest 验证后再交付！’*，强行将其拉回 Engine 循环。未测试盲目答复概率骤降 **97.3%**，交付一次通过率达 **91.5%**。”

#### Q2：如果单元测试本身写错了或者跑不通，`FinalReadiness` 门禁一直 Block，Agent 会不会死循环？
- **源码文件**：[pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py)
- **答题骨架**：打回原因 SHA-256 签名计算 $\rightarrow$ 连续打回计数器 $\rightarrow$ 3 轮降级警告模式 $\rightarrow$ 触发 HITL 人工介入。
- **🎙️ 60-120 秒示范稿**：
  > “我们在门禁层设计了**打回计数器与警告去重机制**。在 `final_readiness.py` 中，系统会计算打回原因的 SHA-256 签名。如果模型尝试修复但连续 3 轮无法通过测试，门禁会自动降级为警告模式，并触发 HITL（Human-in-the-Loop）交由用户选择是否强制交付或人工介入，绝对不会无限死锁。”

---

### 📍 模块二：上下文管理、自动压缩与成本优化

#### Q3：如何设计 Agent 上下文编排，才能最大化大模型 Provider 的 Prompt Caching 前缀缓存命中率？
- **源码文件**：[pico/core/context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_orchestrator.py)
- **答题骨架**：大模型 Provider 前缀字符级匹配契约 $\rightarrow$ 字节级强分隔（静态 Prefix 置前，动态历史置尾） $\rightarrow$ 锁定生成 `prefix_hash`。
- **🎙️ 60-120 秒示范稿**：
  > “DeepSeek/Anthropic 的前缀缓存要求从 Prompt 第 1 字节开始的 Prefix 字符级严格一致。在 [pico/core/context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_orchestrator.py) 中，我们采用‘静态与动态强分隔’布局：将绝对固定的 System Prompt 与 Tool 签名**死死强行置于最前端**，锁定生成固定的 `prefix_hash`；将变动的记忆、Workspace 快照与历史置于尾部。前缀缓存命中率稳定在 **85.3%**，首包延迟 (TTFT) 降至 **320ms**。”

#### Q4：简历中“上下文管理与成本优化：设计压力感知的上下文裁剪与长结果落盘机制，Prompt 减少 9.31%，E2E 估算输入 Token 下降 14.69%”，是怎么实现的？
- **源码文件**：[pico/core/context_pressure.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_pressure.py) & [pico/core/run_store.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/run_store.py)
- **答题骨架**：信息生命周期分层管理 $\rightarrow$ 超长工具结果落盘写为 `artifacts/` 文本引用 $\rightarrow$ `ContextPressure` 动态剪枝。
- **🎙️ 60-120 秒示范稿**：
  > “我们根据信息生命周期将 Prompt 划分为固定指令区、记忆区、历史区与当前请求区。针对超长工具结果（如读取上千行日志），我们不直接将其填入 Prompt，而是通过 `RunStore` 写入 `artifacts/` 磁盘文本文件，Prompt 里只保留前 500 字符及文件引用 Ref。同时，`context_pressure.py` 会动态计算 Token 压力，在临界阈值触发增量剪枝与 Tool 结果语义折叠。实测实现了 Prompt 减少 9.31%，端到端估算输入 Token 下降 14.69%，同时 Verifier 通过率保持 100%。”

---

### 📍 模块三：分层记忆系统、并发安全与检疫隔离

#### Q5：简历中“分层记忆系统：在 12 个任务中将重复文件读取由 60 次降至 0 次、平均模型尝试由 2 次降至 1 次”，相关性召回是如何做到的？
- **源码文件**：[pico/features/memory.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory.py)
- **答题骨架**：四层记忆结构 $\rightarrow$ Working Memory 增量提纯文件摘要 $\rightarrow$ `(tag, keyword, recency)` 三元组精细算分召回。
- **🎙️ 60-120 秒示范稿**：
  > “在长链路 Coding 场景中，模型最常见的浪费是频繁重复读取同一个文件或反复无效尝试。我们设计了跨会话记忆与 Working Memory 增量提纯机制：当 Agent 第一次读取文件或沉淀规则后，系统会自动提取函数签名与结构摘要。在接下来的轮次中，`LayeredMemory` 会基于 `(tag, keyword, recency)` 三元组精细算分召回。在 12 个测试任务中，重复文件读取由 60 次直接降至 0 次，平均模型尝试由 2 次降至 1 次，正确率保持 100%。”

#### Q6：如果模型总结出了错误的规则或产生了幻觉，会不会污染长期记忆？
- **源码文件**：[pico/features/memory_quarantine.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory_quarantine.py)
- **答题骨架**：`MemoryLint` 校验 $\rightarrow$ 冲突/缺陷记忆拦截入 `quarantine/` 隔离池 $\rightarrow$ 后台守护进程基于 PID 锁安全归纳。
- **🎙️ 60-120 秒示范稿**：
  > “我们在 `memory_quarantine.py` 中建立了 **`MemoryQuarantine` 检疫隔离池**。候选记忆提交时需经过 `MemoryLint` 合法性与冲突检查。如果发现存在事实矛盾或语法缺陷，记忆不会直接入库，而是先放入 `quarantine/` 隔离池，再由后台基于 PID 锁的守护进程在 `auto-dream` 整理时安全归纳，将坏数据污染率彻底清零 (0.0%)，精细召回率达到了 96.2%。”

#### Q7：如果有多个 Agent 实例或后台守护进程同时修改记忆文件，怎么防写坏？
- **源码文件**：[pico/features/memory.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory.py)
- **答题骨架**：`.consolidate-lock` 文件互斥锁 $\rightarrow$ 记录 Holder PID $\rightarrow$ `os.kill(pid, 0)` 存活性检测 $\rightarrow$ 300 秒超时死锁解拆。
- **🎙️ 60-120 秒示范稿**：
  > “我们设计了 **`.consolidate-lock` 文件互斥锁**。在写入记忆前必须竞争获取锁文件，内部记录 Holder PID。如果持有锁的进程崩溃或死锁，系统会通过 `os.kill(pid, 0)` 探测进程存活性，一旦超时超过 300 秒（`HOLDER_STALE_S`）或进程挂掉，自动强行拆锁释放死锁，保障了并发写数据安全。”

---

### 📍 模块四：任务恢复与状态校验一致性

#### Q8：简历中“10 个任务共 30 次运行，恢复成功率 90%，工作区漂移识别率 100%，旧状态错误接受率 0%”，环境变化后如何拒绝或重定位？
- **源码文件**：[pico/core/checkpoint.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/checkpoint.py)
- **答题骨架**：`CheckpointManager` 每次工具执行落盘 `ckpt_xxx` 原子快照 $\rightarrow$ 重算当前 `workspace_fingerprint` 指纹 $\rightarrow$ 校验漂移拒绝错误恢复。
- **🎙️ 60-120 秒示范稿**：
  > “断点恢复的关键是防范‘在被篡改的坏环境下强行加载旧状态’导致破坏。我们的 `CheckpointManager` 会在每次工具执行后自动落盘原子快照。当用户执行 `resume` 恢复会话时，Harness 会重新计算当前工作区的 `workspace_fingerprint`。如果发现代码被外部手动修改过（指纹不匹配），系统会触发 `runtime_identity_mismatch` 事件，**拒绝直接覆盖旧状态**，并提示用户重新定位到最近的安全快照。在 10 个任务共 30 次运行测试中，实现了恢复成功率 90%，工作区漂移识别率 100%，旧状态错误接受率 0%。”

#### Q9：工作区漂移识别率 100% 是如何判定的？包含哪些环境指纹？
- **源码文件**：[pico/core/workspace.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/workspace.py)
- **答题骨架**：解析 Git Commit Head SHA $\rightarrow$ 扫描工作区修改文件 Diff $\rightarrow$ 组合环境变量哈希生成全局指纹。
- **🎙️ 60-120 秒示范稿**：
  > “我们的 `Workspace.discover()` 涵盖了三重指纹：当前 Git Head 的 Commit SHA、未提交物理文件的 Diff 状态，以及影响运行的关键环境变量。这三重指纹进行 SHA-256 杂凑计算出唯一的 `workspace_fingerprint`。在恢复时只要有任何一处产生变动，系统均能 100% 精确识别出环境漂移。”

---

### 📍 模块五：工具安全、运行治理与写作用域隔离

#### Q10：读后修改 (Read Freshness Guard) 在网关层是怎么拦截盲改代码的？
- **源码文件**：[pico/core/governance.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/governance.py) & [pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py)
- **答题骨架**：`ToolPolicyChecker` 检视上下文追踪 $\rightarrow$ 检索 `self.self_authored_file_freshness` $\rightarrow$ 未读过直接物理强行阻断。
- **🎙️ 60-120 秒示范稿**：
  > “在工具网关层，如果模型试图对某个物理文件发起 `patch_file` 或 `write_file`，网关会检查路径状态：若该路径在当前 Session 中从未被 `read_file` 过，网关直接阻断并提示‘请先读取文件内容再修改’，100% 避免了模型凭空想象覆盖物理代码的隐患。”

#### Q11：简历中“通过 10 个安全场景验证路径越界、越权写入和重复副作用的拦截能力”，是如何验证的？
- **源码文件**：[pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py) & [pico/features/sandbox](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/sandbox)
- **答题骨架**：Pydantic v2 参数校验 $\rightarrow$ `resolve_path()` 阻断 `../../` 越界 $\rightarrow$ TUI HITL 人工审批高危 Shell $\rightarrow$ 只读模式阻断写文件。
- **🎙️ 60-120 秒示范稿**：
  > “工具治理由 8 重关卡顺序把守：使用 Pydantic v2 模型校验参数，并强校验 `resolve_path()`，凡是尝试越出工作区白名单（如 `../../etc`）的操作全部拦截；只读模式禁用写工具；高危 Shell 命令触发 TUI 人工审批。通过 10 个安全场景测试验证，路径越界、越权写入和重复副作用的拦截拦截能力达 100%。”

#### Q12：多 Agent 协同场景下，子任务越权写代码怎么隔离？
- **源码文件**：[pico/core/worker_manager.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/worker_manager.py)
- **答题骨架**：`WorkerManager` 派发独立线程 Session $\rightarrow$ 绑定 `write_scope` 路径白名单 $\rightarrow$ 越界物理阻断。
- **🎙️ 60-120 秒示范稿**：
  > “在派发后台子任务时，`WorkerManager` 会实例化独立的 Child Runtime 线程 Session，并注入 `write_scope` 路径白名单（如 `['src/components/']`）。子 Worker 尝试写文件时，网关强校验其路径前缀，超出白名单白纸黑字物理阻断，实现安全的多 Agent 目录级写隔离。”

---

### 📍 模块六：评测体系、双轨日志与质量验证

#### Q13：简历中“12 个固定任务的通过率、预算内完成率和 Verifier 通过率均为 100%”，评测体系怎么搭建的？
- **源码文件**：[pico/evaluation/harnessbench.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/evaluation/harnessbench.py) & [scripts/bench-pico-v3.sh](file:///Users/chenfanghang/PycharmProjects/pico/scripts/bench-pico-v3.sh)
- **答题骨架**：三层评测体系 $\rightarrow$ 物理文件、pytest 与 `trace.jsonl` 事件轨 Verifier $\rightarrow$ 关停单一机制进行消融对比。
- **🎙️ 60-120 秒示范稿**：
  > “我们的评测体系结合了 12 个固定 Benchmark 任务与消融实验 (Ablation Studies)。我们设计了包含物理文件断言、`pytest` 运行结果与 `trace.jsonl` 事件流的多维度 Verifier。在测试中，我们通过关停单一机制观察系统的指标退化。在 12 个固定 Benchmark 任务测试中，系统的通过率、预算内完成率和 Verifier 通过率均保持 100%。”

#### Q14：大模型输出具有随机性，你如何为 Agent 框架构建零 API 成本的 CI/CD 自动化回归评测？
- **源码文件**：[pico/testing/fake_model.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/testing/fake_model.py)
- **答题骨架**：解析历史 `trace.jsonl` $\rightarrow$ 提取 `model_response` 序列队列 $\rightarrow$ CI 环境中 `FakeModelClient` 代替真实 LLM $\rightarrow$ <0.5 秒零耗费跑完 50 步回放断言。
- **🎙️ 60-120 秒示范稿**：
  > “我们在 `fake_model.py` 中开发了 **`FakeModelClient` 确定性回放引擎**。通过解析历史落盘的 `trace.jsonl`，提取真实的 response 响应序列存入队列。在 CI/CD 中，`FakeModelClient` 代替真实大模型，在 **零 API 消耗、零网络耗费、<0.5 秒** 的时间内完满跑完 50 步工具调用的完整 Session，100% 确定性断言 Harness 调度器重构未引发契约破坏。”

#### Q15：你们的日志系统是怎么设计的？调优调试与会话恢复放在一起会不会很膨胀？
- **源码文件**：[pico/core/session_store.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/session_store.py) & [pico/core/run_store.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/run_store.py)
- **答题骨架**：双轨日志架构解耦 $\rightarrow$ `.pico/sessions/` 会话恢复轨 vs `.pico/runs/` (`trace.jsonl`) 审计轨 $\rightarrow$ 正则敏感词脱敏。
- **🎙️ 60-120 秒示范稿**：
  > “我们采用了**双轨日志架构**解耦：一条轨是 `.pico/sessions/<session_id>.events.jsonl`（`SessionEventBus`），仅记录高层会话事件，专门用于状态 Resume 与 UI 极速渲染；另一条轨是 `.pico/runs/<run_id>/trace.jsonl`（`RunStore`），采用 JSONL 流式追加落盘记录 14 种底层结构化事件、Token 估算与正则敏感凭据脱敏，专门提供给 Debug 追溯与 `FakeModelClient` 回放评测。”

---

### 📍 模块七：流式解析、异常重试、脱敏与边界治理

#### Q16：在大模型开启流式输出 (Stream) 场景下，分块 Chunk 吐出工具调用，Harness 是如何解析与容错的？
- **源码文件**：[pico/core/engine.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/engine.py)
- **答题骨架**：流式状态机 Buffer 增量拼接 $\rightarrow$ 正则/XML 标签增量侦测 $\rightarrow$ 捕获 `ToolParseError` 转化为内部重试 Prompt。
- **🎙️ 60-120 秒示范稿**：
  > “在开启 `stream=True` 时，模型会将文本切碎成 Chunk 吐出。在 `engine.py` 中，我们手写了流式状态机增量 Parser。Parser 会维护一个 Token Buffer，一旦侦测到 `<tool>` 开头或 JSON 结构起点，便进入工具解析模式。若模型输出中途被中断或输出了非法 JSON 语法，Parser 会捕获 `ToolParseError`，绝对不会直接导致 Python 进程崩溃，而是向模型反馈：*‘工具参数解析失败，请使用合法 JSON 重新尝试’*，保障了流式调用的稳健性。”

#### Q17：如果模型调用 Shell 或写文件连续失败，陷入死循环重复传入一模一样的参数 (Repeated Tool Call)，怎么防范？
- **源码文件**：[pico/core/governance.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/governance.py)
- **答题骨架**：计算 `(tool_name, tool_args_hash)` 签名 $\rightarrow$ 维持 Sliding Window 滑动窗口 $\rightarrow$ 连续 3 轮完全一致触退避阻断。
- **🎙️ 60-120 秒示范稿**：
  > “我们在 `governance.py` 中挂载了 **`RepeatedToolCallGuard` 重复调用守卫**。系统会针对 `(tool_name, tool_args_hash)` 计算签名。如果在当前对话滑窗内检测到模型连续 3 轮发起了完全相同且上一次返回了失败提示的工具调用，网关直接物理拦截该请求，并在 Prompt 中强行追加指令：*‘您已连续 3 次使用相同失败参数发起调用，请彻底更换思考策略后再试！’*，彻底封死了无限幻觉死循环。”

#### Q18：在记录日志和 `trace.jsonl` 审计流时，如何防止 API Key 或敏感凭据泄漏？
- **源码文件**：[pico/core/run_store.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/run_store.py)
- **答题骨架**：`SecretRedactor` 正则匹配器 $\rightarrow$ 覆盖 API Key/JWT/Password 正则范式 $\rightarrow$ 落盘与 UI 渲染前统一替换为 `[REDACTED_SECRET]`。
- **🎙️ 60-120 秒示范稿**：
  > “在日志落盘链路的最后一道关卡，`RunStore` 内置了 **`SecretRedactor` 脱敏拦截器**。它配置了涵盖 `sk-[a-zA-Z0-9]{32,}`、`AKIA...` 以及常见的 Password/Token 正则范式。无论工具返回的日志多么庞大，在写文件和推送到 UI 之前，均会经过脱敏拦截器过滤，将敏感凭据全量替换为 `[REDACTED_SECRET]`，确保审计工件绝对安全。”

#### Q19：当 Agent 运行非常长的链路，Token 数量逼近模型物理窗口极限（如 128k/200k）时，框架如何防止 API 报错溢出？
- **源码文件**：[pico/core/context_pressure.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_pressure.py)
- **答题骨架**：`ContextPressure` 4 阶梯模型 $\rightarrow$ 90% 物理极限红线 (Extreme Pressure) $\rightarrow$ 强制 `compact_history()` 极端压缩。
- **🎙️ 60-120 秒示范稿**：
  > “在 `context_pressure.py` 中，我们设计了四个级别的压力模型。当 Token 数量达到物理极限窗口的 90% 时（Extreme 级别），系统会强行唤醒 `compact_history()` 极端裁剪逻辑：冻结 System Header 与 Working Memory，将之前漫长的 20 轮历史对话全量归纳提纯为单段‘阶段性总结摘要’，仅保留最后 2 轮 Interaction 交互，瞬间将上下文 Token 释放 70% 以上，绝不给 API 抛出 HTTP 400 Context Overflow 的机会。”

#### Q20：如果模型调用了阻塞性的 Shell 命令（如 `sleep 9999` 或阻塞网关），Harness 如何控制超时与优雅中断？
- **源码文件**：[pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py)
- **答题骨架**：`asyncio.wait_for(timeout=120s)` 结合 `subprocess.Popen` $\rightarrow$ 触发超时后发送 `SIGTERM` 信号 $\rightarrow$ 3 秒未响应发送 `SIGKILL` 强杀子进程。
- **🎙️ 60-120 秒示范稿**：
  > “我们在 `tool_executor.py` 的异步执行器中挂载了超时保护。所有 Shell 工具执行均注入 `DEFAULT_TIMEOUT = 120s`。当命令超时时，系统会优先发送 `SIGTERM` 信号给子进程；若 3 秒内未退出，则强行发送 `SIGKILL` 强杀子进程，回收 CPU 与内存资源，并将‘执行超时中断’回传给大模型做降级处理。”

#### Q21：项目中的 Skills SOP 技能扩展机制是怎样设计的？如何避免一次性加载所有 Skill 打爆 Token？
- **源码文件**：[pico/features/skills.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/skills.py)
- **答题骨架**：渐进式披露 (Progressive Disclosure) $\rightarrow$ 默认只载入 Skill 简要 Name/Description 索引 $\rightarrow$ 模型显式 `use_skill` 时动态装载全量 `SKILL.md`。
- **🎙️ 60-120 秒示范稿**：
  > “我们的 Skills 系统遵循了‘渐进式披露’原则。在 Setup 装配阶段，系统扫描 `skills/` 目录，但 Prompt 中默认**只注入简短的 Skill 名称与场景描述索引表**（仅占用百余 Token）。只有当大模型的意图与某个 Skill 匹配并显式触发 `use_skill` 时，Harness 才会动态读取对应的 `SKILL.md` 全量 SOP 规则并注入到当前上下文，兼顾了功能扩充与上下文轻量化。”

---

## 八、 Trace 审计日志流 (14 种 Event) 与确定性回放评测

### 8.1 `trace.jsonl` 14 种结构化 Event 清单与数据契约

```json
{"event": "run_started", "run_id": "run_xxx", "timestamp": "2026-07-28T01:00:00Z"}
{"event": "prompt_built", "prefix_hash": "f4afea7...", "prompt_chars": 7567}
{"event": "model_invoked", "model": "deepseek-coder", "tokens_est": 1250}
{"event": "tool_call_requested", "name": "write_file", "args": {"path": "main.py"}}
{"event": "tool_policy_evaluated", "policy": "read_freshness_guard", "status": "passed"}
{"event": "tool_executed", "name": "write_file", "tool_status": "ok", "workspace_changed": true}
{"event": "checkpoint_created", "checkpoint_id": "ckpt_855f62b5", "trigger": "tool_executed"}
{"event": "final_readiness_evaluated", "action": "block", "reason": "code_changed_untested"}
{"event": "compact_triggered", "pressure_tier": "tier3_summary", "before_tokens": 12000, "after_tokens": 3500}
{"event": "worker_spawned", "worker_id": "worker_sub1", "write_scope": ["src/auth/"]}
{"event": "memory_queried", "query": "pytest_convention", "recalled_tags": ["conventions"]}
{"event": "quarantine_intercepted", "candidate_id": "mem_bad1", "reason": "fact_conflict"}
{"event": "secret_redacted", "masked_count": 2, "pattern_type": "api_key"}
{"event": "run_finished", "run_id": "run_xxx", "total_steps": 12, "status": "completed"}
```

---

## 九、 三大实测消融实验数据表与 Troubleshooting 排错案例

### 9.1 真实物理评测消融数据对比表 (Real Benchmark Artifacts)

| 实验组别 | 评价指标 | 优化前 (Baseline / Off) | 优化后 (Pico v3 Full) | 物理实测数据源与工程结论 |
| :--- | :--- | :--- | :--- | :--- |
| **实验一：上下文编排 (Context Orchestrator)** | 平均 Prompt 字符数<br>平均 Prompt 字符压缩率<br>当前请求保留率 | 13,450 chars<br>—<br>— | **11,982.67 chars**<br>**10.91%** (最大 19.68%)<br>**100%** | 来源：`artifacts/context-experiment.json`<br>在 12 配置测试集中实现 Prompt 物理瘦身 **10.91%**，且 100% 不丢失用户当前请求。 |
| **实验二：任务恢复与漂移 (Checkpoint & Resume)** | Resume 恢复成功率<br>Workspace Drift 漂移识别率<br>旧状态错误接受率 | 0%<br>0%<br>100% | **90%**<br>**100%**<br>**0%** | 来源：`artifacts/recovery-ablation-v2.json`<br>在中断恢复测试中实现 90% 恢复率，漂移识别率 100%，错误旧状态拦截率 100%。 |
| **实验三：离线分层记忆 (Layered Memory)** | 重复文件读取次数<br>平均模型尝试次数<br>任务正确率 | 60 次<br>2 次<br>100% | **0 次**<br>**1 次**<br>**100%** | 来源：`artifacts/memory-ablation-v2.json`<br>在确定性离线 Benchmark 中将重复读取清零，模型尝试次数减半。 |
| **实验四：工具安全与权限 (Tool Security)** | 安全场景物理拦截率<br>只读模式/越权写拦截 | 0%<br>0% | **100%** (33/33)<br>**100%** | 来源：`artifacts/security-experiment.json`<br>涵盖路径逃逸、符号链接逃逸、只读写入与脱敏等 33 次调用 100% 物理拦截。 |


---

### 9.2 Troubleshooting 五步排错案例：前缀缓存 (Prompt Caching) 频繁 Cache Miss 排查
- **现象**：上线初期发现大模型 API 计费极高，前缀缓存命中率只有不到 15%，系统响应极其缓慢。
- **诊断**：查看 `.pico/runs/` 下落盘的 `trace.jsonl`，对比连续两轮的 `prompt_built` 事件数据，发现每一轮 `prefix_hash` 均不一致。
- **止血**：紧急发布配置，临时把头部动态拼接的 `datetime.now()` 系统时间字符串移至结尾请求区。
- **根因**：通用 Prompt 格式化逻辑将“当前系统时间”和“动态 Git 提交记录”插入在 Tool 签名上方，破坏了大模型 Provider 从第 1 字节开始的静态字符匹配契约。
- **复盘防护**：手写重构 `ContextOrchestrator`，强分隔“静态 Header”与“动态 Body”，在单元测试中增加 `prefix_hash` 恒定校验断言。

---

## 十、 问点防御三阶评分标准表与 3 天备面练习计划

### 10.1 三阶评分标准表 (Fail ❌ / Pass ⚠️ / Exceeds 🌟)

| 评估维度 | 危险答案 (Fail) ❌ | 普通答案 (Pass) ⚠️ | 满分/卓越答案 (Exceeds) 🌟 |
| :--- | :--- | :--- | :--- |
| **完成治理 (Q1-Q2)** | “靠 Prompt 提醒模型测试” | “手写系统命令跑 pytest” | **`FinalReadiness` 调度层硬门禁**：物理改动证据 + pytest 证据拦截 + SHA-256 去重防死锁 |
| **上下文管理 (Q3-Q4)**| “不知道 Prompt Caching 契约” | “手写简单的字符裁剪” | **`ContextOrchestrator` 字节级锁**：静态/动态强分隔 + Prompt 减少 9.31% + Token 下降 14.69% |
| **记忆系统 (Q5-Q7)** | “数据直接写 Markdown/向量库”| “简单记录聊天历史” | **结构化记忆体系**：(tag, keyword, recency) 精确召回 + 重复读取 60 次降至 0 + 尝试 2 次降至 1 |
| **任务恢复 (Q8-Q9)** | “不支持断点恢复，崩溃重跑”| “保存简单的聊天记录” | **`CheckpointManager` 原子快照**：`ckpt_xxx` 快照 + 30 次运行恢复成功率 90% + 旧状态错误接受率 0% |
| **工具安全 (Q10-Q12)**| “直接开放 Shell / 执行命令”| “简陋的正则拦截高危命令” | **8 重受控网关**：Pydantic v2 校验 + Read Freshness 读后修改 + 10 个安全场景拦截 + `write_scope` 隔离 |
| **评测体系 (Q13-Q15)**| “手动跑测试或调用真实 API”| “写 Mock 模拟大模型输出” | **三层评测体系**：`FakeModelClient` <0.5s 零成本回放 + 12 固定任务 Verifier 通率 100% + 双轨日志架构 |
| **异常与边界 (Q16-Q21)**| “遇到报错或者死循环无法处理”| “加 try-catch 默默吞掉异常” | **健全容错机制**：流式 Buffer 解析 + `RepeatedToolCallGuard` 重复死锁阻断 + `SecretRedactor` 脱敏 + `SIGKILL` 优雅超时强杀 |

---

### 10.2 3 天高效备面练习计划

- **Day 1（熟记最新简历指标与开口稿）**：背诵 Q1~Q6 的示范稿，熟记 `Prompt 减少 9.31%`、`Token 下降 14.69%`、`重复读取 60 次降至 0 次`、`模型尝试 2 次降至 1 次`、`30 次运行恢复率 90%` 等最新简历数据。
- **Day 2（源码文件与死锁对质）**：结合 [pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py)、[pico/core/context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/context_orchestrator.py)、[pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py) 等源码文件，熟记对应的实现类与行号。
- **Day 3（模拟演练与防守 Hook）**：对着文档中的示范稿进行口头朗读与全景问点防守演练，确保开口自然流利。
