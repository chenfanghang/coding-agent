# Chapter 1：设计哲学与架构选型（ADR）

> [返回全景导航](./00_架构与技术设计白皮书全景导航.md) · [下一章：总体架构与两条主线生命周期](./02_总体架构与两条主线生命周期.md)

> **本章回答三个问题**
>
> 1. Pico 为什么不是一个普通的 ReAct Demo？
> 2. 为什么将上下文、工具权限和完成判断放在模型外？
> 3. 文件化记忆、自研 Harness 和本地持久化分别付出了什么代价？

---

## 1.1 系统定位

Pico v3 是运行在本地代码仓库中的 Coding Agent Harness。模型负责理解任务和提出下一步动作；Harness 负责组装上下文、暴露工具、校验副作用、保存任务状态、判断是否可以结束并记录运行证据。

它重点解决四类工程问题：

| 问题 | 当前机制 | 主要实现 |
| :--- | :--- | :--- |
| 长会话上下文持续增长 | 稳定前缀、分区编排、压力感知压缩、长结果落盘 | `pico/core/context_orchestrator.py`、`context_pressure.py`、`context_manager.py` |
| 模型过早宣布完成 | 基于 TaskState 证据的 Final Readiness | `pico/core/final_readiness.py`、`final_readiness_tools.py` |
| 跨会话信息重复读取或过期 | 分层记忆、相关性排序、新鲜度校验与候选记忆检疫 | `pico/features/memory.py`、`memory_lint.py`、`memory_quarantine.py` |
| 工具调用产生不可控副作用 | 参数校验、权限检查、工具策略、工作区快照与审计事件 | `pico/core/tool_executor.py`、`permissions.py`、`tool_policy.py` |

## 1.2 ADR-001：采用轻量自研 Harness

> **状态：Accepted**

**背景**：项目需要在本地仓库中控制 Prompt 布局、工具副作用、恢复状态和完成条件。

**决策**：使用 Python 构建轻量运行时，不把核心控制循环绑定到通用 Agent 框架。

**理由**：

1. 上下文区块的顺序和稳定性可由 `ContextOrchestrator` 直接控制。
2. 工具权限、TaskState 和完成门禁位于模型外，失败原因可记录和复现。
3. 运行时组件较少，适合本地 CLI/TUI，并便于针对 Coding 场景调整。

**边界**：这不是对 LangGraph、LangChain 等框架的普遍否定。通用框架适合快速组合生态组件；Pico 的选择来自当前项目对运行时控制和可审计性的侧重。

## 1.3 ADR-002：面向缓存的上下文布局

> **状态：Accepted**

**背景**：动态信息若混入 Prompt 前部，会降低相同前缀被 Provider 复用的机会；历史和工具结果持续追加也会推高输入规模。

**决策**：将相对稳定的系统指令与工具描述放在前部，将记忆、工作区状态、历史和当前请求按生命周期置于后部；根据上下文压力逐级缩减动态区。

**理由**：

- 稳定区与动态区分离，便于观测 `prefix_hash` 和 Provider 返回的缓存用量。
- 当前请求、安全约束和关键工作状态具有更高保留优先级。
- 超长工具结果可写入 RunStore 工件，Prompt 中保留摘要和引用。

缓存是否命中仍由 Provider 的具体实现决定。Pico 负责提供缓存友好的布局和采集遥测，不承诺任意 Provider 都会命中。

## 1.4 ADR-003：采用文件化分层记忆

> **状态：Accepted**

**背景**：本地 Coding Agent 既需要保存跨会话事实，也需要让用户能够查看、修改和清理这些状态。

**决策**：使用 Markdown/JSON 文件保存分层记忆，并通过标签、关键词、时间与新鲜度信号进行相关性排序；高风险或冲突候选先经过 Lint 与检疫判断。

**理由**：

- 对路径、符号名和配置项等精确字符串友好。
- 数据可查看、可 Diff，故障排查不依赖专用数据库客户端。
- 与 Workspace 文件新鲜度结合后，可以降低继续使用过期摘要的风险。

**边界**：文件化检索不等于语义检索，也不能保证每次精确召回。项目当前选择可解释的排序机制；未来若语义召回收益明确，可以作为补充而非互斥替代。

## 1.5 ADR-004：将工具治理放在模型外

> **状态：Accepted**

**背景**：仅靠 Prompt 提醒无法可靠阻止路径越界、未读先改、只读模式写入或 Worker 越权。

**决策**：所有工具调用统一经过 `run_tool()`，依次完成工具查找、参数与工作区校验、重复调用检查、权限检查、策略检查、执行和证据记录。

**理由**：

- Pydantic v2 负责值级 Schema 校验，Workspace-aware 校验负责路径和文件状态。
- `PermissionChecker` 管理只读、审批和 `write_scope`；`ToolPolicyChecker` 管理先读后改等使用策略。
- 风险工具执行前后采集工作区快照，用于识别影响路径和部分成功。

**后果**：每个新 Tool 都必须接入 Schema、Profile、Permission、Policy 和审计链路；扩展成本更高，但不会形成绕过统一治理的“特殊工具”。

## 1.6 ADR-005：以证据驱动完成判断

> **状态：Accepted**

**背景**：模型的自然语言“已完成”不能证明工具执行成功、必需工件存在或验证已经运行。

**决策**：在最终答复前调用 `evaluate_final_readiness()`，根据 TaskState 中的原因集合和配置模式返回 `allow`、`warn`、`remind` 或 `block`。

**模式语义**：

- `off`：不根据 readiness 原因干预。
- `warn`：记录并提示，但不阻断。
- `soft`：首次以运行时提醒要求继续处理，相同原因再次出现时避免重复提醒。
- `strict`：存在 hard 原因时阻断；非 hard 原因仍可警告放行。

门禁检查的是实际 `readiness_reasons`，不是“凡改代码必跑 pytest”的单一规则；可接受的验证证据和必需工件由 TaskState 与相关工具判断。

**后果**：交付判断可以从 Prompt 约定提升为 Runtime 决策；代价是 Readiness Reason 必须持续维护，并通过 `warn`、`soft`、`strict` 分级平衡安全和可用性。

## 1.7 ADR-006：审计与恢复状态使用文件落盘

> **状态：Accepted**

**背景**：Pico 面向本地单工作区，需要让运行状态可以直接检查、迁移和复算，同时避免引入数据库服务。

**决策**：会话、任务状态、Trace、报告和大型工具结果分别保存为 JSON、JSONL、Markdown 或文本工件。

**收益**：格式可直接检查，便于原子写入、故障恢复和实验复算。

**代价**：文件化状态不天然提供数据库级事务、复杂查询和多机一致性，因此 Pico 当前更适合本地单工作区使用。

## 1.8 决策汇总

| 决策 | 选择 | 得到的能力 | 主要代价 |
| :--- | :--- | :--- | :--- |
| Runtime | 轻量自研 Harness | 控制循环、工具与状态高度可定制 | 需要自行维护生态集成与运行协议 |
| Context | 稳定区与动态区分离 | 可观测前缀、按压力渐进治理 | Prompt 布局变化需要兼容与回归 |
| Memory | 文件化分层记忆 | 可读、可编辑、适合精确字符串 | 复杂语义检索能力有限 |
| Tool | 模型外治理 | 权限、策略和副作用可审计 | 工具扩展必须同步 Schema 与 Profile |
| Completion | Evidence-based Readiness | 避免只相信模型的完成声明 | 证据规则过严可能产生阻断 |
| Persistence | JSON / JSONL / Markdown | 易调试、易复算、适合本地恢复 | 不适合直接承担分布式一致性 |

## 1.9 源码阅读入口

```text
pico/core/runtime.py
  -> pico/core/engine.py
  -> pico/core/context_orchestrator.py
  -> pico/core/tool_executor.py
  -> pico/core/final_readiness.py
  -> pico/core/runtime_checkpoints.py
  -> pico/core/run_store.py
```

建议先顺着主链路阅读，再进入 `pico/features/memory.py` 和 `pico/core/worker_manager.py` 等专项模块，避免只看单个类而失去运行上下文。

## 1.10 本章小结

Pico 的主要架构选择可以概括为：**让模型保留开放式推理能力，把权限、状态、证据和停止条件收回 Harness**。这也是后续上下文治理、恢复、多 Agent 和分层评测能够统一起来的前提。

### 面试表达

> **30 秒讲法**：Pico 的核心不是重新训练模型，而是在模型外实现一层 Coding Agent Harness。模型负责理解任务和提出动作，Runtime 负责上下文、权限、状态、完成门禁与审计。我选择轻量自研，是因为这些控制面需要统一演进和精确评测；文件化状态则更适合本地单工作区的可检查与可恢复需求。

**项目推演题（非真实面经）**：为什么不直接使用 LangGraph 或现成 Coding Agent？为什么安全和完成判断不能只写进 Prompt？文件化状态与数据库方案各有什么代价？

**回答边界**：不要说 Pico 全面优于通用框架。重点说明自研范围集中在 Harness 控制面，以及它为本项目带来的可观测性和实验可控性。

> **延伸练习**：[Pico 完整高频面试题与参考答案](../面试高频实战/Pico完整高频面试题与参考答案.md)

---

> [返回全景导航](./00_架构与技术设计白皮书全景导航.md) · [下一章：总体架构与两条主线生命周期](./02_总体架构与两条主线生命周期.md)
