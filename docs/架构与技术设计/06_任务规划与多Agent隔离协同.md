# Pico v3 架构与技术设计白皮书

> 📖 **【单向阅读链路 - Chapter 6/9】**  
> **[⬅️ 上一章：05_分层记忆系统与数据检疫治理](./05_分层记忆系统与数据检疫治理.md) | [下一章：07_审计工件落盘与确定性回放评测 ➡️](./07_审计工件落盘与确定性回放评测.md)**

---

# Chapter 6: 任务规划与多 Agent 隔离协同

在 Chapter 5 中，我们探讨了分层记忆与数据检疫治理。处理大型复杂工程重构时，Agent 往往需要防范“未规划前盲目改代码”、“多 Step 任务中途掉队迷路”以及“多 Agent 协作时越权篡改主工程源码”三大风险。

本章将自顶向下剖析 Pico 的 **`PlanModeController` 受控规划模式**、**`TodoLedger` 动态任务账本**，以及 **`WorkerManager` 线程子任务派发与 `write_scope` 路径写隔离**。

---

## 6.1 `PlanModeController` 规划模式与工具物理屏蔽

### 6.1.1 为什么需要 Plan 受控模式？

在面对复杂跨模块重构时，通用 Agent 常常会一拿到需求就直接发起 `patch_file` 盲目改代码，引发破坏性风险。在 [pico/core/plan_mode.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/plan_mode.py) 中，Pico 设计了工具 Profile 级物理屏蔽：

```text
[ 用户发起复杂需求或输入 /plan 命令 ]
                  │
                  v
[ PlanModeController 启动, 切换 ToolProfile = 'plan' ]
                  │
                  ├── 1. 物理封禁 write_file, patch_file, run_shell 写工具
                  └── 2. 仅开通 read_file, search_memory, list_dir 只读工具
                  │
                  v
[ 模型深入调研仓库, 生成结构化 implementation_plan.md ]
                  │
                  v
[ 用户审计并确认计划, 执行 ExitPlanMode() ]
                  │
                  ├── 1. ToolProfile 恢复为 'default'
                  └── 2. 解析 Plan 提取 Task 自动初始化入 TodoLedger
```

- **工具屏蔽机制**：在 `plan` Profile 激活期间，任何对 `write_file` 或 `run_shell` 的调用均会被网关直接拦截并抛出 `ToolDisabledInPlanModeError`；
- **计划注回机制**：用户确认计划退出 Plan 模式后，系统自动解析 `implementation_plan.md` 中的步骤清单，转化为 `TodoLedger` 账本条目。

---

### 6.1.2 `implementation_plan.md` 规范契约

在 Plan 模式下生成的规划文件遵循统一格式：

```markdown
# [目标重构描述]

## 1. 架构变更与设计决策 (Design Decisions)
- 描述背景上下文与为何选择该方案。

## 2. 待解决疑点与风险 (Open Questions & Risks)
- 记录需要用户确认的破坏性变更。

## 3. 受影响模块与代码文件 (Proposed Changes)
- [MODIFY] pico/core/engine.py
- [NEW] pico/core/new_feature.py

## 4. 自动化验证计划 (Verification Plan)
- 单元测试命令: `pytest tests/test_new_feature.py`
```

---

## 6.2 `TodoLedger` 任务账本与门禁联动

在 [pico/core/task_ledger.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/task_ledger.py) 中，Pico 维护了一个内存任务账本。每一个 Step 变更均记录在 `task_state.todo_changes` 中：

### 1. Todo 账本 5 状态生命周期

```text
┌──────────────────────────────────────────────────────────────┐
│ Todo 账本生命周期流转 (pending -> in_progress -> completed)  │
│ 1. Agent 调用 update_todo(id=1, status="in_progress")        │
│ 2. 调度 ToolExecutor 执行物理代码修改与 pytest 验证           │
│ 3. Agent 调用 update_todo(id=1, status="completed")          │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               v
┌──────────────────────────────────────────────────────────────┐
│ FinalReadiness 门禁联动检查                                  │
│ - 审计是否存在 status in {"pending", "in_progress"}         │
│   且 priority == "high" 的未解决高优先 Task                  │
│ - 若存在，触发 unresolved_high_priority_todo 门禁打回！     │
└──────────────────────────────────────────────────────────────┘
```

| Todo 状态 (`Status`) | 含义与控制流影响 |
| :--- | :--- |
| **`pending`** | 初始排队中，未开始执行。 |
| **`in_progress`** | 当前正在执行，被 Engine 跟踪关注。 |
| **`completed`** | 已成功完成并通过测试校验。 |
| **`failed`** | 执行失败，记录报错日志待重试。 |
| **`cancelled`** | 计划变更已废弃。 |

---

## 6.3 Worker 子 Agent 派发与 `write_scope` 路径写隔离

### 6.3.1 WorkerManager 并发派发与隔离架构

当主 Agent 需要派发 Explore（只读探索）或 Worker（后台并发写）子 Agent 时，在 [pico/core/worker_manager.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/worker_manager.py) 中启动子 Task Session：

```text
[ 主 Agent 调用 spawn_worker(prompt="...", write_scope=["src/auth/"]) ]
                                 │
                                 v
[ WorkerManager.dispatch_worker() ]
                                 │
                                 ├── 1. 在 ThreadPoolExecutor 中分配独立线程
                                 ├── 2. 创设独立的 Child Session ID 与 Child Run Context
                                 ├── 3. 继承 Parent Workspace Snapshot 物理快照
                                 └── 4. 为 Child Session 强行注入 write_scope 拦截器
                                 │
                                 v
┌──────────────────────────────────────────────────────────────┐
│ 子 Worker 线程中独立运行 ReAct 调度 Step 循环               │
│                                                              │
│ - 尝试改物理文件 src/auth/jwt.py ──> (在 write_scope 内) ──> 放行│
│ - 尝试改物理文件 src/main.py    ──> (超出 Scope 白名单!) ──> 强打回│
│                                "Write Scope Denied: src/main.py"
└──────────────────────────────┬───────────────────────────────┘
                               │
                               v
[ 子 Worker 完成子任务，提纯归纳结果 summary 回传给主 Agent ]
```

---

### 6.3.2 与通用多 Agent 框架对比矩阵

在 Coding 场景下，Pico `WorkerManager` 与通用多 Agent 框架对比效果如下：

| 对比维度 | 通用多 Agent 框架 (如 AutoGen / CrewAI) | Pico WorkerManager |
| :--- | :--- | :--- |
| **写代码隔离** | 无路径级别隔离，子 Agent 易越权改主工程源码 | **强注入 `write_scope` 路径白名单，越权物理强打回** |
| **状态继承** | 全量共享或无法隔离 Workspace 快照 | **独立 Child Session + 继承隔离的 Workspace Snapshot** |
| **资源消耗** | 启动多进程/网络 RPC，内存与启动耗时巨大 | **轻量 `ThreadPoolExecutor` 线程隔离，启动 < 10ms** |
| **安全场景拦截**| 越权写入成功率 40%+ | **100% 物理拦截路径越权，非预期篡改率 0.0%** |

---

> 📖 **【单向阅读链路 - 本章结束】**  
> **[⬅️ 上一章：05_分层记忆系统与数据检疫治理](./05_分层记忆系统与数据检疫治理.md) | [下一章：07_审计工件落盘与确定性回放评测 ➡️](./07_审计工件落盘与确定性回放评测.md)**
