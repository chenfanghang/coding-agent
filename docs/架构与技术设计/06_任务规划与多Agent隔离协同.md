# Chapter 6：任务规划与多 Agent 隔离协同

> [上一章：分层记忆系统](./05_分层记忆系统与数据检疫治理.md) · [下一章：任务恢复与状态一致性](./07_任务恢复与状态一致性.md)

> **本章关注**：复杂任务如何先调研再实施、进度如何持续可见、并行子任务如何限制写入范围。

---

## 6.1 Plan Mode

`PlanModeManager` 将 Session 的 `runtime_mode` 切换为 `plan`，并刷新工具 Profile 与稳定前缀。在该模式下：

- 可以检查 Workspace；
- 写操作只能指向当前 `.pico/plans/` 下的活动计划工件；
- 只允许启动只读 `Explore` 子 Agent；
- 最终答复前必须已经写入非空计划工件。

```text
enter_plan_mode
  -> 保存 plan_path
  -> Tool Profile = plan
  -> 调研与更新 Todo
  -> 写入活动计划工件
  -> exit_plan_mode
  -> Tool Profile = default
```

退出 Plan Mode 不会自动解析计划并创建 Todo；Todo 由 Agent 通过显式工具维护。文档不应把尚未实现的自动转换写成当前能力。

## 6.2 TodoLedger

`TodoLedger` 位于 `pico/core/todo_ledger.py`，状态保存在 Session 中，并将变更记录到当前 TaskState 和事件总线。

| 状态 | 含义 |
| :--- | :--- |
| `pending` | 尚未开始 |
| `in_progress` | 正在执行 |
| `done` | 已完成 |
| `blocked` | 因明确条件无法继续 |

优先级为 `low`、`normal`、`high`。Todo 是任务控制面，不代表对应代码已经通过测试；完成证据仍由 TaskState 和 Final Readiness 判断。

### Todo、TaskState 与 Plan 的区别

| 对象 | 回答的问题 | 是否跨轮次保存 |
| :--- | :--- | :---: |
| Plan Artifact | 准备如何实施，风险和验证方案是什么 | 是 |
| TodoLedger | 当前有哪些工作项，各自状态是什么 | 是 |
| TaskState | 这次请求实际做了什么，有哪些证据和停止原因 | 单次 Run 持久化 |

Plan 是方案，Todo 是控制面，TaskState 是运行事实。三者相互关联，但不能互相代替。

## 6.3 Worker 生命周期

`WorkerManager` 管理 `worker` 与 `Explore` 两类子 Agent。存在 `model_client_factory` 时可使用后台线程执行，否则同步运行。每个 Worker 拥有 Child Runtime、独立状态和可选写入范围。

```text
主 Agent
  -> spawn(description, prompt, type, write_scope)
  -> Child Runtime
       |-- Explore：只读探索
       `-- worker：在权限允许时写入
  -> 状态 / 结果 / 通知回传主 Agent
```

支持的生命周期操作包括创建、继续、停止请求、通知回收和 Runtime 关闭。线程提供调度并发，不等同于进程级安全边界。

## 6.4 `write_scope` 的实际语义

`write_scope` 是 Worker 写工具的路径白名单，由 Child Runtime 的权限检查执行。它用于限制 Agent 通过注册工具发起的写入，但不能表述为完整操作系统沙箱：如果未来引入绕过工具层的写入口，仍需额外隔离。

当前 Worker 采用两层约束：

1. `worker` Tool Profile 不暴露 `run_shell`、协调器工具、模式切换工具和交互工具；
2. `write_file` 与 `patch_file` 继续由 PermissionChecker 检查目标路径是否位于 `write_scope`。

`Explore` 始终只读；普通 Worker 如果没有提供 `write_scope`，Child Runtime 同样会进入只读状态。因此 Worker 不能通过 `run_shell` 绕过路径白名单，但这仍是当前 Tool Profile 下的应用层保证，不是独立进程或文件系统 Namespace。

安全结论应通过越权用例验证，不能依据代码分支直接推导“100% 拦截”。当前场景集和失败边界见 [运行时安全与记忆治理评测](../评测体系/07_运行时安全与记忆治理评测.md)。

## 6.5 协同设计中的取舍

| 选择 | 优点 | 限制 |
| :--- | :--- | :--- |
| Child Runtime | 状态和工具面可以独立配置 | 仍共享本地工作区，需要作用域治理 |
| 后台线程 | 实现简单，可复用当前进程依赖 | 不提供进程级故障与资源隔离 |
| `Explore` / `worker` 两类角色 | 权限语义清晰 | 不承担复杂组织层级建模 |
| 显式 `write_scope` | 路径边界可审计 | 只约束注册工具路径，不能替代 OS Sandbox |
| 通知队列 | 主 Agent 可异步接收完成结果 | 仍需处理停止与 Runtime 关闭 |

## 6.6 源码索引与本章小结

- `pico/core/plan_mode.py`：运行模式和计划工件约束。
- `pico/core/todo_ledger.py`：Todo 状态与事件。
- `pico/core/worker_manager.py`：Worker 创建、继续、停止与通知。
- `pico/core/worker_runtime.py`：Child Runtime 装配。
- `pico/core/worker_execution.py`：子任务执行过程。
- `pico/core/permissions.py`：Plan Mode 与 `write_scope` 权限判定。

Pico 的多 Agent 设计优先追求**边界清晰和执行可追踪**，而不是最大化角色数量。是否值得派发 Worker，应由任务是否可独立、上下文是否可隔离以及并行收益是否真实决定。

### 面试表达

> **30 秒讲法**：复杂任务先通过 Plan Mode 和 TodoLedger 显式拆分，再把可独立子任务交给 Child Runtime。Explore 只读，普通 Worker 通过 Tool Profile 缩小工具面，并由 `write_scope` 限制写入路径；执行结果和通知回传主 Agent，由主 Agent 负责最终整合与验收。

**项目推演题（非真实面经）**：什么时候值得派发 Worker？主 Agent 与 Child Runtime 共享什么、隔离什么？Worker 能否通过 Shell 绕过 `write_scope`？后台执行如何停止和回收？

**回答边界**：当前使用后台线程与应用层工具隔离，不是进程级沙箱；不要把“多 Agent”描述成角色越多越先进，应强调任务独立性、写入边界和可追踪性。

> **延伸练习**：[Pico 完整高频面试题与参考答案](../面试高频实战/Pico完整高频面试题与参考答案.md)

---

> [上一章：分层记忆系统](./05_分层记忆系统与数据检疫治理.md) · [下一章：任务恢复与状态一致性](./07_任务恢复与状态一致性.md)
