# Pico v3 高频面试题与硬核参考答案全集

> 📌 **使用说明**：本文档整理了面试 Pico AI Agent Harness 过程中的高频问点与硬核 STAR 参考答案。遇到深入技术点追问时，可直接跳转底层的 6 大解耦专属架构文档库：

- 🧠 **分层记忆系统**：[docs/记忆/README.md](../记忆/README.md) (Working / Durable / Quarantine / Retrieval / Auto-Dream)
- ⚡ **上下文编排系统**：[docs/上下文/README.md](../上下文/README.md) (Prefix Lock 85.3% Caching / Section Budget / Pressure Tier)
- 🛠️ **受控工具网关**：[docs/工具调用/README.md](../工具调用/README.md) (8 重受控关卡 / Read Freshness / Path Escape / Repetition)
- 🔄 **任务恢复系统**：[docs/任务恢复/README.md](../任务恢复/README.md) (5 大 Resume 状态矩阵 / Workspace Drift / Re-anchoring)
- 🛡️ **完成门禁系统**：[docs/完成门禁/README.md](../完成门禁/README.md) (Final Readiness 5 大检查项 / Block 打回强提醒 / Hooks)
- 🤖 **多 Agent 协同**：[docs/多Agent协同/README.md](../多Agent协同/README.md) (Explore 与 Worker 分工 / write_scope 100% 写入隔离)

---

# Pico AI Agent 工程师面试全景实战指南（简历追问全量版）

> 📌 **文档定位**：围绕简历中的“本地 Coding Agent Harness”项目，按“简历映射—开口稿—架构主线—源码级攻防—实验数据—故障复盘—训练计划”组织。本文只使用当前源码和 2026-07-29 DeepSeek V4 Flash 全量评测能够支撑的事实，不把受控实验扩大为生产结论。
>
> 🎯 **使用方式**：初面优先练习 1 分钟开口稿与 Q1-Q21；二面重点准备 Q22-Q43、实验数据和失败复盘；面试前使用末尾速记卡完成指标口径检查。

---

## 📚 目录导航

- [一、简历原文与技术证据映射](#一-简历原文与技术证据映射)
- [二、1 分钟与 3 分钟项目开口稿](#二-1-分钟与-3-分钟项目开口稿)
- [三、项目定位与 Agent Runtime 全链路](#三-项目定位与-agent-runtime-全链路)
- [四、全景高频问点深度攻防](#四-全景高频问点深度攻防)
- [五、最新实验数据与指标防守](#五-最新实验数据与指标防守)
- [六、失败案例与 Troubleshooting](#六-失败案例与-troubleshooting)
- [七、评分标准与 3 天训练计划](#七-评分标准与-3-天训练计划)

---

## 一、 简历原文与技术证据映射

### 1.1 📄 当前简历项目原文

**本地 Coding Agent Harness**　`2026.02 - 至今`

- **项目概况**：开发面向代码仓库长链路任务的本地 Coding Agent Harness，在模型外统一执行控制、上下文治理、跨会话记忆与任务恢复，形成可控、可恢复的 Agent Runtime。
- **Agent Harness**：构建状态驱动的 Agent 控制循环，统一模型调用、工具执行与会话持久化，并通过完成门禁与结构化 Trace 实现长链路任务的可控执行。
- **上下文与成本优化**：设计面向缓存的稳定前缀与压力感知渐进压缩机制，前缀缓存命中率达 85.3%，真实模型实验中 Prompt 平均压缩 9.48%，任务正确率保持 100%。
- **分层记忆系统**：构建跨会话分层记忆与自动沉淀机制，支持相关性召回、事实更新与新鲜度校验，Memory Challenge 准确率达 94.55%，Evidence Recall@K 达 100%。
- **可靠性与安全治理**：通过任务恢复、状态漂移识别与工具权限隔离保障执行安全，恢复成功率 90%，漂移识别率 100%，错误接受率 0%。
- **评测体系**：构建确定性 Benchmark、消融对照与真实任务验收体系，覆盖任务正确性、执行成本、恢复能力与安全边界，真实场景通过率达 98%。

### 1.2 🧭 简历指标、实验口径与源码映射

| 简历技术点 | 最新可讲数据 | 样本与口径 | 核心源码 |
| :--- | :--- | :--- | :--- |
| Agent Harness | 12 项任务通过率、Verifier、预算内完成率均为 100% | 固定 Harness Benchmark | `pico/core/engine.py`、`task_state.py`、`final_readiness.py` |
| 稳定前缀 | 前缀缓存命中率由12.1%提升至85.3% | 独立缓存实验，架构文档记录；最新全量报告未重跑 | `context_orchestrator.py`、`context_sections.py` |
| 压力感知压缩 | Prompt 平均压缩 9.48%，最高 19.57%，正确率 100% | 12 配置 × 5 次 × 2 变体，共 120 次 DeepSeek Live 调用；字符指标 | `context_pressure.py`、`context_manager.py`、`turn_history.py` |
| 分层记忆 | Accuracy 94.55%，Recall@K 100%，Precision@K 87.23% | 55 个 Memory Challenge | `pico/features/memory.py`、`evaluation/memory_agent_eval.py` |
| 任务恢复 | 恢复 90%，漂移识别 100%，错误接受 0% | 11 个任务定义 × 3 次；开启与关闭各 33 次 | `runtime_checkpoints.py`、`workspace.py` |
| 安全与验收 | 49/50，通过率 98% | 50 个真实使用方式的端到端场景，失败项为 S45 | `tool_executor.py`、`permissions.py`、`evaluation/metrics.py` |

### 1.3 🛡️ 面试中必须守住的四条边界

1. 9.48% 是 **Prompt 字符压缩率**，不是 Provider 实际 Token 或账单成本下降。
2. 85.3% 来自独立前缀缓存实验，不能说成最新 120 次上下文矩阵的聚合结果。
3. Memory Challenge 的 Recall@K 100% 不等于答案 100% 正确；最终准确率是 94.55%。
4. 98% 是 50 个评测场景中的 49/50，不是线上用户成功率或生产 SLA。

---

## 二、 1 分钟与 3 分钟项目开口稿

### 2.1 🎙️ 1 分钟开门见山版

> “Pico 是我开发的本地 Coding Agent Harness，目标是把大模型不确定的工具意图放进一条可控、可恢复、可审计的执行链路。它主要解决代码仓库长链路任务中的上下文膨胀、工具越权、跨会话状态丢失，以及模型修改代码后未验证就结束的问题。
>
> 在执行层，我实现了状态驱动控制循环，把模型调用、工具执行、Checkpoint 和完成门禁统一到 Agent Runtime；在上下文侧，通过稳定前缀、压力感知裁剪、长结果落盘和历史压缩控制 Prompt；在记忆侧，通过分层召回、事实更新和新鲜度校验支持跨会话复用；最后用结构化 Trace 和 Verifier 建立评测证据链。
>
> 最新 DeepSeek V4 Flash 实验中，真实上下文矩阵 Prompt 平均压缩 9.48%，正确率保持 100%；Memory Challenge 准确率 94.55%，Recall@K 100%；50 个真实场景通过 49 个。”

### 2.2 🎙️ 3 分钟深度项目版

> “这个项目的出发点不是再封装一个模型 API，而是解决 Agent 从 Demo 走向真实代码仓库时的运行时问题。普通 ReAct 循环可以让模型决定下一步工具，但它很难保证工具是否越权、状态是否过期、任务是否真的完成，以及中断后能不能安全继续。
>
> 所以我把 Pico 设计成模型外的 Harness。请求进入后，Runtime先装配模型、工具、Session、Memory、Skills 和权限配置；ContextOrchestrator按稳定前缀、记忆、历史和当前请求组装 Prompt；模型只输出工具意图或 Final；工具调用必须经过格式、Schema、重复调用、权限、路径和使用策略检查；每次执行都会更新 TaskState、Trace 和 Checkpoint；模型尝试结束时，还要由 Final Readiness 检查修改、验证、必需产物和失败证据。
>
> 上下文部分采用四级压力治理：低压观察，中压裁剪，较高压力将长结果落盘，极高压力再压缩历史；记忆部分区分 Session、工作记忆和 Durable Memory，并通过来源、新鲜度和敏感信息规则控制写入与召回；恢复时不仅恢复聊天记录，还会校验 Workspace 和 Runtime Identity。
>
> 评测上我分成确定性 Benchmark、机制消融、真实模型矩阵和端到端场景。12 个固定任务三项指标均为 100%；真实上下文矩阵共 120 次调用，Prompt 平均压缩 9.48%且正确率保持100%；55 个 Memory Challenge 准确率94.55%；恢复实验报告90%成功率，漂移识别100%；50个场景通过率98%。这些分层实验让我能区分‘机制合同是否成立’和‘真实模型效果是否成立’。”

---

## 三、 项目定位与 Agent Runtime 全链路

### 3.1 🎯 Pico 解决的核心问题

```text
模型能力：提出下一步、理解语义、生成代码
                     |
                     v
Harness 治理：权限、状态、预算、恢复、证据、完成判定
                     |
                     v
外部世界：文件、Shell、测试、工作区、持久化数据
```

模型负责“想做什么”，Harness负责“能不能做、做完发生了什么、是否允许结束”。

### 3.2 🔄 一次请求的完整生命周期

```text
[1 Runtime 装配]
模型 / Provider / Workspace / Tool Profile / Session / Memory / Skills
        |
        v
[2 Prompt 构建]
稳定前缀 -> 工作记忆 -> Skills -> 相关记忆 -> 历史 -> 当前请求
        |
        v
[3 模型决策]
<tool> 工具意图  或  <final> 最终回答
        |
        +------------------------------+
        |                              |
        v                              v
[4 工具治理与执行]               [7 完成就绪门禁]
解析 / Schema / 重复调用          修改、测试、产物、失败证据
权限 / 路径 / Tool Policy              |
        |                         allow / warn / block
        v                              |
[5 状态与证据持久化]                   v
History / TaskState / Trace       [8 交付或继续循环]
Checkpoint / Workspace 变化
        |
        v
[6 下一轮 Prompt]
```

### 3.3 🗂️ 三类状态不能混为一谈

| 状态层 | 解决的问题 | 主要载体 |
| :--- | :--- | :--- |
| Session | 多轮对话和用户交互如何延续 | Session events、History |
| Run / TaskState | 本次任务执行到哪里、为何停止 | `task_state.json`、`report.json` |
| Trace | 每个模型、工具和治理决策如何发生 | `trace.jsonl` |

---

## 四、 全景高频问点深度攻防

### 📍 模块一：项目定位与个人贡献

#### 🧩 实现总览：我如何把基础 ReAct 循环演进为 Coding Agent Harness

Pico最初解决的是“模型能够调用工具，但任务过程不可控”的问题。基础ReAct可以完成“模型决策—工具执行—结果回传”，但状态、权限和完成条件主要依赖Prompt，遇到跨文件修改、测试失败、中断恢复或工具部分成功时，很难判断任务到底进行到哪一步。我因此没有继续堆叠Prompt，而是把模型外的执行治理抽成Harness：由模型产生意图，由Runtime掌握状态和副作用，由Verifier判断结果。

我的实现顺序是先建立最小模型—工具循环，再逐步补上TaskState、Tool Gateway、Checkpoint、Context、Memory、Final Readiness和Trace。各模块不直接彼此耦合，而是通过结构化状态和事件协作：Engine只负责循环与状态转换；工具层负责权限和执行；Context与Memory负责模型输入；RunStore负责证据；Final Readiness负责结束判定。这样底层Provider或编排框架可以替换，Harness合同仍然保留。

```text
基础 ReAct
Model -> Tool -> Observation -> Model
                     |
                     v
Pico Harness
Model Intent
    -> Runtime State
    -> Tool Governance
    -> Workspace Side Effect
    -> Checkpoint / Trace
    -> Final Readiness
```

| 核心层 | 我的实现重点 | 解决的问题 |
| :--- | :--- | :--- |
| Engine | 状态驱动循环与显式终态 | 模型何时继续、失败或停止 |
| Runtime | 统一装配Provider、工具、会话和配置 | 避免能力散落 |
| Governance | 权限、路径、策略和重复调用门禁 | 防止越权与重复副作用 |
| State & Evidence | TaskState、Checkpoint、Trace、Report | 可恢复、可审计、可评测 |

> 🎯 **这一模块的面试主线**：我不是重新发明ReAct，而是为ReAct补上真实代码任务需要的运行时治理层。

#### Q1：请用一分钟介绍 Pico。

- **源码文件**：`pico/core/runtime.py`、`engine.py`、`context_manager.py`、`pico/features/memory.py`。
- **答题骨架**：项目定位 → 四类痛点 → 核心机制 → 两到三个指标 → 当前边界。
- **🎙️ 60-120 秒示范稿**：
  > “Pico 是我开发的本地 Coding Agent Harness，面向需要多轮检索、修改、测试和恢复的代码仓库长链路任务。它和普通工具调用Demo最大的区别，是把控制权放在模型外：模型只负责提出下一步，Harness负责上下文预算、工具权限、TaskState、Checkpoint、完成门禁和Trace审计。这样既能保留模型处理开放问题的能力，又能让每次副作用和停止原因可验证。最新DeepSeek上下文矩阵共120次调用，Prompt平均压缩9.48%且目标任务正确率保持100%；55个Memory Challenge准确率94.55%、Recall@K为100%；50个真实使用场景通过49个。当前项目已具备完整运行和评测闭环，但我不会把这些受控结果扩大成生产SLA。”

#### Q2：为什么要自己实现 Harness，LangGraph 或 DeepAgents 不能解决吗？

- **源码文件**：`pico/core/engine.py`、`runtime.py`、`tool_executor.py`。
- **答题骨架**：认可通用框架 → 指出 Coding Runtime 特有治理 → 说明可替换边界。
- **🎙️ 60-120 秒示范稿**：
  > “我不是认为LangGraph或DeepAgents能力不足，而是它们解决的层次和Pico不同。通用框架擅长图编排、工具注册和Checkpointer，适合快速搭建Agent流程；Pico要解决的是本地Coding Runtime的治理问题，比如模型修改已有文件前是否读过最新内容、Worker能写哪些目录、Workspace漂移后能否继续、长工具结果如何落盘，以及模型说完成时是否真的有测试和产物证据。这些约束依赖真实文件和运行状态，单靠Prompt或增加几个图节点不够。我的取舍是把Harness做成独立控制层，同时保留清晰接口；如果未来换成LangGraph做底层编排，上下文、权限、门禁、Trace和评测合同仍然可以复用，而不是把系统绑死在自研循环上。”

#### Q3：Pico 和普通 ReAct Agent 的本质区别是什么？

- **源码文件**：`pico/core/engine.py`、`task_state.py`、`tool_executor.py`。
- **答题骨架**：ReAct负责决策 → Harness负责约束和事实 → 强调不是新推理算法。
- **🎙️ 60-120 秒示范稿**：
  > “ReAct主要描述模型侧的推理循环，也就是根据观察决定下一步行动，再根据工具结果继续推理。它的问题是权限、状态和停止条件经常隐含在对话文本里，模型一句‘已经完成’就可能结束。Pico保留ReAct的灵活决策，但在外部增加确定性Harness：工具调用先经过Schema、路径、权限和重复调用检查；执行后写入TaskState、Trace和Checkpoint；输出Final时再检查修改、测试、Todo和必需产物。举例来说，模型可以决定运行pytest，但不能伪造pytest已经通过。我的理解是，ReAct回答‘下一步想做什么’，Harness负责‘这一步能否执行、执行后发生了什么、任务是否允许结束’，两者是推理层与运行治理层的关系。”

#### Q4：什么任务才算代码仓库长链路任务？

- **源码文件**：`pico/core/engine.py`、`runtime_checkpoints.py`、`final_readiness.py`。
- **答题骨架**：多轮模型 + 多工具 + 外部状态变化 + 中断恢复。
- **🎙️ 60-120 秒示范稿**：
  > “我把同时具备多轮模型决策、多次工具调用和外部状态变化的任务定义为长链路任务。一个典型例子是先搜索入口和测试，读取多个相关文件，形成修改计划，Patch实现，运行pytest，再根据失败日志定位第二处问题，最后确认报告或其他必需产物已经落盘。难点不只是步骤数量，而是前一步会改变Workspace并影响后一步：工具可能超时但已经写入部分内容，用户可能中断任务，外部也可能在恢复前修改文件。只保存聊天历史无法判断这些事实。因此Pico需要显式记录TaskState、文件变化、Todo、验证信号和Checkpoint，并在恢复与完成时重新校验。我所说的长链路强调的是状态依赖和副作用，而不是简单把对话轮数拉长。”

#### Q5：这个项目中哪些模块是你独立设计和实现的？

- **源码文件**：`pico/core/`、`pico/features/`、`pico/evaluation/`及对应Git提交记录。
- **答题骨架**：问题定义 → 核心模块 → 评测 → 复用能力 → Git证据。
- **🎙️ 60-120 秒示范稿**：
  > “这是个人工程项目，但我仍然会按可核验模块说明贡献，而不是笼统说‘全部都是我做的’。我重点设计和实现了状态驱动Engine、ContextOrchestrator与Compact、分层Memory、Checkpoint/Resume、Tool Gateway、Final Readiness，以及确定性Benchmark、消融和Live评测链路。Provider HTTP协议、基础文件工具和部分框架思想属于适配或参考，我不会包装成原创。为了证明贡献，我会从问题、设计、代码和实验四层回答，例如上下文模块可以对应`context_pressure.py`的阈值、`context_manager.py`的分区预算，以及120次DeepSeek矩阵的结果；恢复模块可以对应11个任务、每个3次的消融产物。面试前我还会准备3到5个代表Commit，确保每项主张都能落到具体实现。”

#### Q6：Pico目前处于什么阶段，有真实用户吗？

- **证据文件**：`README.md`、`pico/evaluation/`、最新DeepSeek全量评测报告。
- **答题骨架**：个人工程项目 → 已完成的验证 → 不等于生产平台。
- **🎙️ 60-120 秒示范稿**：
  > “Pico目前是一个可运行、具备完整测试和评测体系的个人Agent工程项目，不是公司内部平台，也还没有大规模真实用户。它已经完成513项Pytest回归、12项固定Harness Benchmark、DeepSeek Live矩阵、50个真实使用方式的端到端场景、Gate8和业务Dogfood，所以可以证明系统不是只停留在架构图或Demo。但这里的‘真实场景’是指使用真实Provider、CLI、Skills、恢复和安全路径进行验收，不等于线上自然流量，更不能直接称为生产SLA。现阶段项目价值是验证Agent Harness的工程机制和评测方法；下一阶段才是扩大到更多真实仓库、跨天任务和外部用户，观察这些受控指标是否还能保持。”

---

### 📍 模块二：Agent Harness 控制循环

#### 🧩 实现总览：状态驱动控制循环如何运行

控制循环是Pico的主干。我把一次任务拆成“构建Prompt、请求模型、解析意图、治理工具、执行副作用、持久化证据、判断是否继续”七个阶段。模型输出不会直接映射成函数调用，而是先解析成`tool`、`tools`、`retry`或`final`四类结果；工具必须通过统一Gateway；Final也必须经过完成门禁。这样模型只能提出候选动作，真正的状态转换始终由Engine控制。

Engine同时维护两类预算：`max_steps`限制真实工具执行次数，`max_steps + 2`限制模型总尝试次数。每次工具执行后都会写TaskState、Trace并创建Checkpoint，因此即使工具部分成功、进程中断或模型没有给出Final，系统仍然知道已经发生了哪些副作用。Session用于跨轮交互，Run用于一次执行，Trace用于过程审计，三者各自承担不同职责。

```text
build_prompt()
      |
      v
model_call() -> parse_output()
      |             |
      |       tool / tools / retry / final
      |             |
      v             v
Tool Gateway     Final Readiness
      |             |
      v             v
Tool Executor   allow / remind / block
      |
      v
TaskState -> Trace -> Checkpoint -> next turn
```

| 组件 | 关键职责 | 主要产物 |
| :--- | :--- | :--- |
| `Engine` | 循环、预算、状态转换 | attempts、tool_steps、stop_reason |
| `model_output` | 解析并归一化模型协议 | tool / retry / final |
| `tool_executor` | 治理与执行工具 | status、error_code、affected_paths |
| `FinalReadiness` | 检查交付证据 | allow、notice、block |
| `RunStore` | 持久化运行证据 | TaskState、Trace、Report、Checkpoint |

> 🎯 **这一模块的面试主线**：状态机的价值不是让流程更复杂，而是让模型错误、工具副作用和任务终态都能被系统解释。

#### Q7：一次用户请求会经过哪些阶段？

- **源码文件**：`pico/core/runtime.py`、`engine.py`、`engine_helpers.py`。
- **答题骨架**：装配 → Prompt → 模型 → 工具网关 → 状态持久化 → 完成门禁。
- **🎙️ 60-120 秒示范稿**：
  > “一次请求进入后，Runtime先装配Provider、模型、Workspace、Tool Profile、Session、Memory、Skills和审批策略，并创建本次Run与TaskState。随后ContextManager按稳定前缀、工作记忆、Skills、相关记忆、历史和当前请求组装Prompt，ContextPressure决定是否需要裁剪或压缩。模型返回后，Parser把结果分成Tool、Tools、Retry或Final。工具意图要依次经过Schema、重复调用、权限、Tool Policy和路径边界，再由Executor执行；结果会写入History、TaskState、Trace和Checkpoint。模型给出Final时，Final Readiness检查修改、测试、Todo、必需产物和失败证据，决定允许、提醒模型继续，或者直接阻断。最终Session用于续接，Report用于聚合，Trace保留完整过程。”

#### Q8：状态机有哪些核心状态，状态如何转换？

- **源码文件**：`pico/core/task_state.py`、`turn_transitions.py`。
- **答题骨架**：运行状态 + 过程计数 + 转移触发条件。
- **🎙️ 60-120 秒示范稿**：
  > “TaskState的终态包括completed、stopped和failed，运行中记录attempts、tool_steps、last_tool、stop_reason、Checkpoint和证据摘要。有效工具批次、格式修复、Provider重试和门禁提醒都会形成continue transition；合法Final进入completed；达到step或retry预算进入stopped；不可恢复的Provider或Runtime错误进入failed。转移依据是结构化事实，不靠模型自然语言声明。”

#### Q9：模型返回工具调用后，Harness如何解析和执行？

- **源码文件**：`pico/core/model_output.py`、`tool_executor.py`。
- **答题骨架**：标签解析 → Payload归一化 → 多层治理 → 执行 → 证据回写。
- **🎙️ 60-120 秒示范稿**：
  > “Pico要求模型返回`<tool>`或`<final>`文本协议。`model_output.py`先扫描一个或多个Tool Block，把JSON或受支持的XML写法统一成`name + args`结构；如果响应为空、JSON非法、缺少标签，或者args不是对象，不会让Python进程直接报错，而是生成明确的Retry Notice，让模型在有限次数内重写。解析成功后，Engine按顺序处理工具批次，每个工具还要经过注册表、参数Schema、重复调用、Permission、Tool Policy和路径边界检查。真正执行后，系统记录tool status、error code、affected paths以及Workspace是否变化，再更新History、TaskState、Trace和Checkpoint。这样解析、授权、执行和持久化是四个独立阶段，任何失败都能定位到明确环节。”

#### Q10：模型失败、工具失败和参数错误分别如何处理？

- **源码文件**：`pico/core/model_errors.py`、`engine_helpers.py`、`tool_executor.py`。
- **答题骨架**：可修复输入错误 → 工具副作用感知 → Provider有限重试 → 失败持久化。
- **🎙️ 60-120 秒示范稿**：
  > “我把失败分成三类处理。第一类是模型格式或参数错误，例如非法JSON、缺少必填字段，这类通常可修复，Engine会返回结构化Runtime Notice，告诉模型具体问题并允许有限次数重写。第二类是工具失败，系统不仅记录退出码和错误摘要，还检查affected paths与Workspace变化；如果命令失败但已经修改文件，会标记partial success，要求先检查Diff，不能假设没有副作用后直接重试。第三类是Provider网络或HTTP错误，只对明确可重试的类型按预算重试，超过限制后把TaskState标记failed。三类失败都会记录attempt、stop reason和Trace，并保存可用Checkpoint，用户能看到失败而不是被隐形重试掩盖。”

#### Q11：什么是完成门禁，为什么不能相信模型自己判断完成？

- **源码文件**：`pico/core/final_readiness.py`、`final_readiness_reasons.py`。
- **答题骨架**：模型的概率判断 → 运行时的确定性证据 → allow/warn/block。
- **🎙️ 60-120 秒示范稿**：
  > “完成门禁解决的是Premature Exit，也就是模型认为自己完成了，但运行事实并不支持。Pico不会看到`<final>`就直接退出，而是检查changed paths、测试或其他verification signal、必需产物、高优Todo、治理拒绝、工具partial success和上下文压缩状态，再返回allow、runtime notice或block。比如模型修改了源码却没跑测试，Prompt里即使写了‘请记得验证’，模型仍可能忽略；而Engine能直接读取本次Run里是否真的出现验证证据。门禁第一次可以提醒模型补救，硬缺失则阻断，并把原因写入Trace。我的设计原则是：模型可以决定如何完成，也可以解释为什么无法完成，但不能用一段自然语言覆盖文件、测试和产物这些确定性事实。”

#### Q12：如果模型一直不满足门禁，会不会死循环？

- **源码文件**：`pico/core/engine.py`、`completion_governance.py`。
- **答题骨架**：工具预算 + 模型尝试上限 + 重复调用拦截 + 可恢复停止。
- **🎙️ 60-120 秒示范稿**：
  > “不会无限循环，因为Pico在模型外设置了多层预算。首先，`max_steps`限制本轮真正执行的工具数量；模型总尝试上限是`max_steps + 2`，额外空间只用于格式修复和最终回答。其次，完全相同的工具名和参数会被重复调用守卫识别，避免模型在同一错误上空转。Provider HTTP层默认最多额外重试2次，Engine也只对特定可恢复错误做有限补偿。达到工具预算后，系统会请求模型生成一段受限的进度总结，明确已完成、未完成和如何继续，并保存Checkpoint；连续格式错误则以`retry_limit_reached`停止。这样系统选择可恢复地停下，而不是为了追求自动完成无限消耗Token或重复副作用。”

#### Q13：Trace记录什么，如何定位一次失败？

- **源码文件**：`pico/core/run_store.py`、`runtime_events.py`。
- **答题骨架**：事件范围 → 定位顺序 → Trace与Report区别。
- **🎙️ 60-120 秒示范稿**：
  > “Trace是每个Run的结构化事件流，采用JSONL追加写入，记录run开始、Prompt构建、模型请求与解析、工具开始与结束、权限决策、记忆召回、Checkpoint、上下文治理和Final Readiness等事件。定位失败时我先看Report和TaskState里的status与stop_reason，判断属于模型、工具、预算、审批还是门禁；然后回到Trace按span顺序检查模型当时看到了什么、解析成了什么调用、工具是否产生Workspace变化，以及Verifier缺少哪条证据。比如S45就是先看到`secret_shaped`拒绝事件，再发现最终存储Verifier失败，从而定位为决策与持久化不一致。Report回答‘结果是什么’，Trace回答‘为什么得到这个结果’，两者分开便于审计和回放。”

---

### 📍 模块三：上下文工程与前缀缓存

#### 🧩 实现总览：面向缓存的分区编排与压力感知压缩

上下文模块同时解决两个问题：一是把最稳定的内容放到Prompt前部，提高Provider前缀缓存复用概率；二是在历史增长时控制输入规模，又不丢失当前请求和执行约束。我没有使用单一滑动窗口，而是先把Prompt拆成Prefix、Memory、Skills、Relevant Memory、History和Current Request六个Section，为每区配置预算、下限、保护状态和缩减顺序。

每轮构建Prompt时，ContextPressure根据实际或估算输入Token计算`pressure_ratio`。低压时保持全量内容；压力升高后依次执行旧历史缩短、长工具结果落盘和结构化历史压缩。长结果不会直接消失，而是写入Artifact，Prompt保留预览、哈希和引用；接近预算上限时才触发Compact或LLM Handoff。Current Request没有普通裁剪预算，工具契约和安全规则则通过稳定Prefix与版本签名维持。

```text
Stable Prefix
  System Rules + Tool Contract + Runtime Mode
                         |
                         v
Memory -> Skills -> Relevant Memory -> History -> Current Request
                         |
                         v
                 ContextPressure
        tier0      tier1      tier2      tier3
       observe ->  snip  ->   prune  -> summary/handoff
                               |
                               v
                     Artifact Preview + Ref
```

| 机制 | 实现方式 | 主要目标 |
| :--- | :--- | :--- |
| 稳定前缀 | 固定顺序、序列化和工具签名 | 提高Prefix Cache复用 |
| 分区预算 | Section budget、floor、protected | 控制裁剪边界 |
| 长结果落盘 | 预览、SHA-256、Artifact引用 | 保留证据而不塞满Prompt |
| 历史压缩 | Deterministic Compact / LLM Handoff | 释放长会话空间 |

> 🎯 **这一模块的面试主线**：先按信息稳定性和生命周期编排，再按压力渐进治理；压缩率必须与正确率、Provider usage分开验证。

#### Q14：什么是缓存友好的稳定前缀？

- **源码文件**：`pico/core/context_orchestrator.py`、`context_sections.py`。
- **答题骨架**：稳定区与动态区分离 → 顺序稳定 → 正确失效。
- **🎙️ 60-120 秒示范稿**：
  > “缓存友好的稳定前缀，本质上是按变化频率重新组织Prompt。Pico把系统规则、工具契约、运行模式和相对稳定的Workspace说明放在最前部，把记忆、历史、实时状态和当前请求放到后部。这样连续轮次只改变后半段，Provider更容易复用相同的前缀Token。实现时不能只保证内容大致相同，还要固定Section顺序、字段序列化和工具签名，避免时间戳、随机遍历顺序或动态统计进入稳定区。另一方面，稳定不等于永不变化：工具集合、运行模式或关键配置改变时，系统必须刷新`prefix_hash`和`prompt_cache_key`，防止错误复用。Pico负责的是前缀布局与失效契约，真正的缓存存储和命中由模型服务商完成。”

#### Q15：85.3%的前缀缓存命中率怎么定义、怎么统计？

- **源码文件**：`pico/core/context_orchestrator.py`、`providers/clients.py`。
- **答题骨架**：Provider缓存字段 → 分子分母 → 对照组 → 数据边界。
- **🎙️ 60-120 秒示范稿**：
  > “命中率应使用Provider返回的实际缓存字段，分子是命中稳定前缀缓存的有效请求，分母是具备可复用前缀并进入统计的请求。独立实验保持任务和模型一致，对照组使用动态头部，命中率12.1%；治理组分离稳定前缀与动态历史，命中率85.3%，提升73.2个百分点，约为原来的7倍。Pico本地负责稳定布局和`prompt_cache_key`，真正的Prefix Cache由Provider执行。”

> ⚠️ **证据边界**：12.1%和85.3%记录在`docs/架构与技术设计/04_上下文编排与前缀缓存优化.md`及审计评测文档中；2026-07-29全量报告没有重新执行该实验，也没有保留可复算的有效请求总数和Provider原始缓存字段。因此可以讲独立实验对照结果，但不能说它来自最新120次上下文矩阵。

#### Q16：压力感知中的“压力”如何计算，分几级？

- **源码文件**：`pico/core/context_pressure.py`。
- **答题骨架**：输入Token / 可用Prompt预算 → 四档阈值 → 对应动作。
- **🎙️ 60-120 秒示范稿**：
  > “Pico不是按历史轮数机械压缩，而是用`pressure_ratio`判断当前输入离Prompt预算还有多远。这个比例等于当前输入Token除以可用Prompt预算，并结合Provider实际usage做校准。低于0.60是`tier0_observe`，保留完整内容；0.60到0.80进入`tier1_snip`，缩短旧轮次和文件快照；0.80到0.95进入`tier2_prune`，进一步裁剪，并把长工具结果替换为Artifact预览、哈希和引用；0.95以上才进入`tier3_summary`，对较早历史做结构化Compact，满足触发条件时再使用LLM Handoff。这样设计是为了让治理成本与压力匹配：短上下文不付摘要成本，中长上下文先做确定性处理，只有接近预算时才调用更强但更贵的压缩机制。”

#### Q17：压缩时先删什么，最后必须保留什么？

- **源码文件**：`pico/core/context_manager.py`、`context_sections.py`、`turn_history.py`。
- **答题骨架**：分区预算 → Reduction order → floor/protected → 当前请求不可普通裁剪。
- **🎙️ 60-120 秒示范稿**：
  > “Prompt按Prefix、Memory、Skills、Relevant Memory、History和Current Request分区。超预算时默认缩减顺序是Relevant Memory、Skills、History、Memory、Prefix，每区都有floor；Current Request没有普通裁剪预算并被标记为protected。长工具结果不直接丢弃，而是保留预览、哈希和Artifact引用；最近轮次和关键下一步优先保留。系统安全规则与工具契约位于受版本签名约束的前缀，不能像普通历史一样无痕删除。”

#### Q18：Prompt平均压缩9.48%是怎么测出来的？

- **源码文件**：`pico/evaluation/metrics.py`、`context_cost.py`。
- **答题骨架**：实验矩阵 → 对照变体 → 指标单位 → 正确率门禁。
- **🎙️ 60-120 秒示范稿**：
  > “9.48%来自DeepSeek V4 Flash真实上下文矩阵，不是离线估算。任务是在Memory Note中放入唯一目标Token，同时加入不同数量的干扰Note和无关历史，要求模型只返回目标Token。实验组合短中长3档历史、低高2档Note、短长2档请求，共12种配置；每种配置重复5次，并分别运行Full与Raw两个变体，因此总计120次Live调用。Raw Prompt平均13474字符，治理后平均12024.92字符，按配置计算的平均压缩率是9.48%，最高19.57%。两组目标Token正确率都为100%，说明当前矩阵中压缩没有破坏任务。但它只覆盖受控事实召回，不足以证明任意长链路代码任务都语义无损。”

#### Q19：如何证明压缩没有丢请求？为什么有场景出现负收益？

- **源码文件**：`pico/core/context_manager.py`、`pico/evaluation/metrics.py`。
- **答题骨架**：结构保护 + Verifier → 100%只代表测试集 → 固定开销解释负值。
- **🎙️ 60-120 秒示范稿**：
  > “我从结构保护和结果验证两层检查压缩是否丢信息。结构上，Current Request没有普通裁剪预算并标记为protected；最近轮次和关键下一步优先保留；长工具结果不是直接删除，而是保留预览、哈希和Artifact引用；安全规则与工具契约位于受版本签名约束的Prefix。结果上，Full与Raw都要通过目标Token Verifier，当前120次调用正确率均为100%。不过我不会把它表述为绝对无损，因为任务分布仍然有限。矩阵最低压缩率是-2.53%，发生在短历史场景：可回收内容很少，但Section标题和编排元数据存在固定成本。更合理的优化是预测可回收字符与固定开销，只有净收益为正时才启用治理。”

#### Q20：为什么9.48%不能直接说成Token或成本下降？

- **源码文件**：`pico/core/context_usage.py`、`pico/evaluation/context_cost.py`、`scripts/run_llm_handoff_benchmark.py`。
- **答题骨架**：字符 ≠ Token ≠ 账单 → 缓存、输出、摘要调用和重试都会影响成本。
- **🎙️ 60-120 秒示范稿**：
  > “不能直接这样说，因为字符、Token和账单是三种不同口径。9.48%计算的是Prompt字符变化，具体会映射成多少Token取决于模型Tokenizer；最终费用还受缓存Token价格、输出Token、额外摘要调用、工具循环和失败重试影响。最新Handoff实验就是一个反例：未缓存输入变化中位数下降15.30%，配置价格中位数下降5.93%，但由于多了一次摘要调用和更多工具步骤，平均总输入Token反而增加4.26%，并出现6个质量退化配对。因此我会分别报告字符压缩、Provider实际usage和配置价格，只有总成本在重复实验中稳定下降且质量门禁不退化，才会写‘成本下降’。目前简历只写Prompt压缩是更准确的。”

#### Q21：LLM Handoff和确定性Compact有什么区别？

- **源码文件**：`pico/core/context_handoff.py`、`compact.py`、`compact_summary.py`。
- **答题骨架**：确定性可复现 vs LLM语义摘要 → 降级路径 → 最新实验边界。
- **🎙️ 60-120 秒示范稿**：
  > “确定性Compact和LLM Handoff解决的都是长历史压缩，但取舍不同。Compact按固定规则选择较早轮次，保留最近交互和关键字段，结果可复现、成本低，失败边界也清楚；缺点是难以理解隐含语义。LLM Handoff会额外调用模型，把目标、约束、涉及文件、已有决策、当前阻塞和下一步压成结构化交接，语义表达更强，但也会产生额外Token，并可能遗漏关键事实。Pico会校验Handoff结构，生成失败或收益不足时回退确定性Compact。最新5个任务、3次重复形成15组有效配对，Handoff Verifier是100%，高于Compact的93.33%，但总输入Token增加4.26%且有6组质量退化，所以目前只能说它有质量潜力，不能宣称稳定降本。”

---

### 📍 模块四：分层记忆与事实治理

#### 🧩 实现总览：跨会话记忆如何写入、召回、更新和失效

记忆模块不是把聊天记录长期保存，而是管理“哪些事实值得跨会话复用，以及这些事实现在是否仍可信”。我按生命周期把数据分成Session History、Working Memory和Durable Memory：Session保留原始交互，Working Memory维护当前任务摘要、文件摘要和Todo，Durable Memory只接收经过筛选的长期约定、决策和项目事实。

写入时，候选事实必须经过来源、长期价值、敏感形态和冲突检查；用户明确事实与已验证工具结果优先，未经证实的模型推断不能直接晋升。召回时基于标签、关键词、来源、新鲜度和时间选择Relevant Memory，并记录selected与rejected evidence。文件型事实绑定路径和内容哈希，Workspace变化后自动标记stale；存在明确更新关系时使用supersede，没有更新关系的冲突则应拒答或重新验证。

```text
User Fact / Tool Evidence / Task Decision
                  |
                  v
          Candidate Extraction
                  |
        +---------+----------+
        |                    |
  Safety / Conflict       Reject / Quarantine
        |
        v
   Durable Memory
        |
        v
Query -> Retrieval -> Freshness -> Relevant Memory -> Prompt
                         |
                         v
                  selected / rejected evidence
```

| 组件 | 实现重点 | 防止的问题 |
| :--- | :--- | :--- |
| Working Memory | 任务与文件状态增量更新 | 反复重建上下文 |
| Durable Promotion | 来源、价值、Secret与冲突检查 | 幻觉和临时状态污染 |
| Retrieval | 标签、关键词、来源、新鲜度排序 | 全量记忆注入 |
| Supersede / Stale | 更新链、文件哈希、Workspace状态 | 使用过期或冲突事实 |

> 🎯 **这一模块的面试主线**：记忆质量不只看“是否召回”，还要看证据是否新鲜、安全，以及模型能否正确使用。

#### Q22：记忆系统分为哪几层，各自存什么？

- **源码文件**：`pico/features/memory.py`、`session_store.py`。
- **答题骨架**：Session历史 → 工作记忆 → Durable Memory → 召回时按需注入。
- **🎙️ 60-120 秒示范稿**：
  > “Pico把记忆按生命周期分成三层。第一层是Session History，保存原始用户、模型和工具交互，主要用于当前会话连续性；第二层是Working Memory，保存任务摘要、文件摘要、Todo、当前阻塞和执行依赖，内容会随着任务进展更新；第三层是Durable Memory，只保存经过筛选的长期约定、关键决策和跨会话可复用事实。这样短期过程不会无差别污染长期知识。召回时系统也不会把所有记忆塞进Prompt，而是根据查询、标签、关键词、来源和新鲜度选择Relevant Memory，并记录选中和拒绝原因。文件型摘要还绑定路径和内容哈希，Workspace变化后旧摘要会标记过期或要求重新读取，从而把‘记住了什么’和‘现在还能不能相信’分开处理。”

#### Q23：什么内容会自动沉淀，如何避免把幻觉写进长期记忆？

- **源码文件**：`pico/features/memory.py`、`memory_lint.py`、`memory_quarantine.py`。
- **答题骨架**：候选来源 → 安全与价值检查 → 统一写入口 → 禁止模型推断直接落库。
- **🎙️ 60-120 秒示范稿**：
  > “自动沉淀不等于把模型最终回答整段写进长期记忆。Pico先从用户明确表达、已验证工具结果和已经执行的任务决策中生成候选，再检查来源、长度、敏感形态、事实冲突和跨会话价值。像API Key、只对当前Run有效的临时状态、未经工具验证的模型猜测，都不应该进入Durable Memory。候选通过后还要保存source、created time和更新关系，便于后续做新鲜度与supersede判断；存在冲突或风险时进入拒绝或隔离路径，而不是直接覆盖旧事实。这里最重要的原则是让工具和用户事实优先于模型总结。S45也说明，仅记录一条拒绝事件还不够，过滤必须收口到统一持久化入口，并由最终存储Verifier再次确认。”

#### Q24：两条记忆冲突时如何选择，新鲜度根据什么判断？

- **源码文件**：`pico/features/memory.py`、`pico/core/workspace.py`。
- **答题骨架**：显式supersede优先 → 来源与时间不能单独决定 → 文件哈希/Workspace状态校验。
- **🎙️ 60-120 秒示范稿**：
  > “冲突处理不能简单采用‘时间最新就覆盖’，因为一条更新更晚的模型总结未必比早期工具证据可靠。Pico优先检查是否存在明确supersede关系；如果用户或可信工具明确更新了旧事实，新条目可以取代旧条目，同时保留更新链。如果两条事实互相矛盾但没有更新关系，正确行为应该是返回unknown或要求重新验证，而不是强行选边。新鲜度也分不同来源：文件型事实绑定路径和内容哈希，内容变化后摘要标记stale；任务恢复还会比较Workspace fingerprint、Schema和Runtime Identity；一般Note则结合来源、时间和状态判断。Memory Challenge中失败的3个案例正是没有supersede关系却选了其中一条，因此后续优化重点是冲突识别和克制回答，而不是继续提高召回数量。”

#### Q25：记忆检索使用关键词、Embedding还是混合召回？为什么不用向量库？

- **源码文件**：`pico/features/memory.py`。
- **答题骨架**：当前数据类型与规模 → 可解释检索 → 向量库边界 → 混合召回演进。
- **🎙️ 60-120 秒示范稿**：
  > “当前实现没有直接引入向量数据库，而是采用可解释的结构化召回。因为Pico保存的主要是项目约定、配置事实、文件摘要和任务决策，这些内容通常包含明确关键词、路径和标签，系统可以综合标签匹配、关键词重叠、来源、新鲜度和时间排序，并把每条记忆为什么选中或拒绝写进Trace。这样调试冲突、过期和敏感信息更直接，也避免在数据量较小时引入Embedding和向量库维护成本。它的边界是同义表达和复杂语义关系召回较弱。规模扩大后，我会采用关键词加Embedding的混合召回，但向量相似度只能负责候选生成，最终仍要经过来源、freshness、supersede和Secret过滤，不能因为语义相似就直接注入Prompt。”

#### Q26：Memory Challenge的55个案例怎么设计？

- **源码文件**：`pico/evaluation/memory_agent_eval.py`。
- **答题骨架**：六类任务 → 四个变体 → 多指标而非单分数。
- **🎙️ 60-120 秒示范稿**：
  > “Memory Challenge不是只测简单事实问答，而是用55个案例覆盖记忆生命周期。具体分为信息抽取9个、知识更新9个、多会话推理9个、时间与冲突推理11个、拒答8个、Agent效率9个。每个Case都定义查询、期望答案、required evidence、禁止使用的记忆以及是否属于Secret、Stale或无答案场景。实验同时运行memory_on、memory_off、naive_recent和unsafe_memory四个变体，用来区分没有记忆、只用最近内容和缺少安全治理的不同基线。最终不只看Answer Accuracy，还看Evidence Recall与Precision、Stale Use、Secret Exposure、False Resume和重复读取。这样可以避免一个总分看起来很高，却掩盖过期事实或敏感信息被使用的问题。”

#### Q27：Evidence Recall@K为100%是什么意思？K是多少，Ground Truth如何标注？

- **源码文件**：`pico/evaluation/memory_agent_eval.py`、`pico/features/memory.py`。
- **答题骨架**：required evidence → selected evidence → Recall与Accuracy分离 → K证据缺口。
- **🎙️ 60-120 秒示范稿**：
  > “Evidence Recall@K评估的是检索阶段，不是最终回答。每个Fixture预先标注`required_evidence_ids`作为Ground Truth，运行时记录`selected_evidence_ids`；Recall的计算是被召回的必要证据数除以必要证据总数。达到100%说明55个案例中，需要的证据都进入了选中集合，但模型仍可能错误筛选、组合或理解，所以最终Answer Accuracy只有94.55%，Precision@K是87.23%。当前产物没有把固定K作为独立字段落盘，实际每个Case选中0到2条证据，因此我不会在面试里编造一个K值。更完整的改进是把retrieval limit和实际selected count都写入评测Schema，确保Recall@K可以从原始产物直接复算。”

#### Q28：Recall@K为100%，为什么仍有3个失败案例？

- **源码文件**：`pico/evaluation/memory_agent_eval.py`、`memory-challenge-v1.json`。
- **答题骨架**：检索成功 ≠ 推理成功 → 三个冲突事实案例 → 正确行为应拒答。
- **🎙️ 60-120 秒示范稿**：
  > “三个失败案例都不是检索漏掉了证据，而是证据使用阶段出现了问题。它们分别询问项目负责人、Runner和报告Owner，每个Case都召回了两条互相冲突的记忆，但两条之间没有明确supersede关系，因此Ground Truth要求回答unknown。系统却分别选择了team alpha、python和runtime team，相当于在证据不足时强行选边。这也解释了为什么Recall@K能达到100%，Answer Accuracy仍只有94.55%：检索已经把冲突双方都找到了，但推理层没有正确执行克制策略。针对这类失败，我不会继续增加召回数量，而是先增加冲突标记、可信来源比较和unknown门禁，并把这3个Case作为固定回归集。”

#### Q29：真实模型接入记忆后，是否减少了重复读取和Token？

- **源码文件**：`pico/evaluation/metrics.py`、`memory_agent_eval.py`。
- **答题骨架**：先给结论 → 离线与Live分开 → 不写未成立收益。
- **🎙️ 60-120 秒示范稿**：
  > “结论是：离线机制收益成立，但真实模型效率收益目前没有成立。受控ScriptedModel中，memory_on把重复读取从60次降到0次，平均模型尝试从2次降到1次，两组正确率都为100%；这证明记忆注入和工具合同可以工作。但在真实DeepSeek矩阵中，memory_on、memory_off和irrelevant三组正确率仍然都是100%，memory_on重复读取67次，反而高于off的58次，平均工具步数也是1.12对0.97。说明模型虽然看到了记忆，仍倾向重新读文件验证。当前汇总也没有提供能支持Token下降的配对结果，所以不能说真实模型已经降本。下一步要让记忆携带来源、文件版本和可信状态，并只在证据缺失、冲突或过期时建议重读，再做Live A/B验证。”

---

### 📍 模块五：任务恢复与运行时安全

#### 🧩 实现总览：如何在环境变化后安全续接任务

任务恢复的难点不是重新加载对话，而是判断中断后的外部世界还能否支持旧计划。Pico在工具执行、上下文缩减和状态变化等关键节点创建Checkpoint，保存目标、下一步、TaskState、Todo、关键文件锚点和Runtime Identity。Resume时重新计算文件freshness、Workspace fingerprint、Schema、工具签名和运行配置，再把恢复状态分成full-valid、partial-stale、workspace-mismatch、schema-mismatch和no-checkpoint。

恢复安全与工具安全共用一个原则：不可信状态不能靠模型解释后绕过。部分陈旧时先重读受影响文件；工作区或运行身份不一致时重新锚定；证据缺失时明确停止。工具侧则通过Tool Profile、Schema、路径解析、Permission/HITL、Tool Policy和Sandbox逐层限制能力，Worker还绑定write scope。所有允许与拒绝都写入治理事件，供恢复、审计和Verifier使用。

```text
Checkpoint
 Goal + Next Step + TaskState + Todo + File Anchors + Runtime Identity
                                 |
                                 v
                         Resume Validation
        +------------+-----------+-------------+-------------+
        |            |                         |             |
   full-valid   partial-stale          workspace/schema   no-checkpoint
        |            |                    mismatch            |
     continue     re-read & reanchor       safe stop        safe stop

Tool Request -> Profile -> Schema/Path -> Permission/HITL
             -> Tool Policy -> Sandbox -> Trace
```

| 机制 | 核心检查 | 输出 |
| :--- | :--- | :--- |
| Checkpoint | 目标、下一步、状态和文件锚点 | 可恢复快照 |
| Freshness | 文件哈希与Workspace Identity | stale / mismatch状态 |
| Permission | 模式、审批、Worker write scope | allow / deny |
| Sandbox | 路径、命令和超时边界 | 受控执行结果 |

> 🎯 **这一模块的面试主线**：恢复成功率不是越高越好；可信时继续、不可信时拒绝，才是可靠恢复。

#### Q30：任务恢复和普通聊天历史恢复有什么区别？

- **源码文件**：`pico/core/runtime_checkpoints.py`、`workspace.py`。
- **答题骨架**：历史文本 vs 外部世界 → Checkpoint + Freshness + Runtime Identity。
- **🎙️ 60-120 秒示范稿**：
  > “普通聊天历史恢复只能重建‘之前说过什么’，但代码任务还依赖文件、工具和Todo这些外部状态。比如上次会话准备Patch某个函数，恢复前文件已经被用户修改，如果只把历史重新塞给模型，旧计划可能覆盖新代码。Pico因此把恢复设计成状态校验：Checkpoint保存目标、下一步、TaskState、Todo、关键文件锚点、最后工具和Runtime Identity；恢复时重新检查Schema、文件freshness、Workspace fingerprint、工具签名和运行配置。完全一致时可以延续，部分陈旧时先重新锚定，不兼容或缺少证据时拒绝直接继续。任务恢复的目标不是尽可能恢复，而是在外部世界仍可信时恢复，并避免重复副作用。”

#### Q31：Checkpoint保存哪些状态，恢复时如何决定下一步？

- **源码文件**：`pico/core/runtime_checkpoints.py`、`task_state.py`、`todo_ledger.py`。
- **答题骨架**：目标/下一步/状态/锚点 → 四种恢复分支。
- **🎙️ 60-120 秒示范稿**：
  > “Checkpoint不是一个简单的聊天摘要，它包含当前目标、建议下一步、关键文件锚点、TaskState、Todo、最后一次工具执行、已知Workspace状态和Runtime Identity。恢复时系统先比较当前环境与Checkpoint，再决定动作：`full-valid`表示依赖仍一致，可以从下一步继续；`partial-stale`表示部分文件变化，需要先读取受影响路径并更新计划；`workspace-mismatch`或运行身份变化会记录mismatch并重新锚定，不能直接复用旧工具意图；Schema版本不兼容时走安全拒绝；完全缺少Checkpoint则返回`no-checkpoint`。因此下一步不是模型从历史里猜出来的，而是旧计划、已完成证据和当前Workspace三者校验后的结果。”

#### Q32：恢复成功率90%的分母是什么，剩余10%为什么失败？

- **源码文件**：`pico/evaluation/metrics.py`、`recovery-ablation-v2.json`。
- **答题骨架**：11任务×3 → 30/33 → 唯一失败类型 → 安全拒绝不是恢复错误。
- **🎙️ 60-120 秒示范稿**：
  > “恢复成功率90%来自受控消融，不是线上会话统计。实验包含11个任务定义，每个重复3次，resume enabled和disabled各运行33次。开启恢复后，10类任务达到预期，逐次结果是30/33，汇总报告按一位小数记录为90%；关闭恢复后，恢复、首动作和Todo连续性指标为0。唯一未恢复的是`schema_mismatch_missing`：系统找不到可信Checkpoint，因此返回`no-checkpoint`，没有根据残留聊天内容猜测旧任务。这个失败不代表系统把正确状态恢复错了，而是面对证据缺失选择安全停止。面试中我会同时说分母、失败Case和0% false accept，避免只报90%让人误以为剩余部分都是程序异常。”

#### Q33：Workspace Drift是什么，为什么识别后不能直接继续？

- **源码文件**：`pico/core/workspace.py`、`runtime_checkpoints.py`。
- **答题骨架**：Checkpoint依赖的外部事实变化 → 旧计划可能产生错误副作用 → 重新锚定。
- **🎙️ 60-120 秒示范稿**：
  > “Workspace Drift指Checkpoint依赖的外部事实已经变化，例如关键文件内容、Workspace fingerprint、工具签名或运行配置与中断时不一致。它危险的地方在于旧计划可能仍然语义合理，但应用对象已经变了：继续执行可能把旧Patch打到新版本文件、重复一次已经发生的副作用，或者错误地把Todo标记完成。Pico在恢复时重算当前身份并与Checkpoint比较，发现partial stale时要求先重读受影响文件，发现Workspace mismatch时记录`runtime_identity_mismatch`并重新锚定，而不是静默继续。恢复实验中构造了6次Workspace mismatch，6次全部识别；这个100%只代表已覆盖的漂移类型，不等于现实中所有环境变化都能被识别。”

#### Q34：工具权限隔离包含哪些层？

- **源码文件**：`pico/core/tool_profiles.py`、`permissions.py`、`tool_policy.py`、`features/sandbox/`。
- **答题骨架**：能力面收缩 → Schema/路径 → Permission/HITL → 使用策略 → Sandbox。
- **🎙️ 60-120 秒示范稿**：
  > “Pico的工具安全不是靠一条危险命令正则，而是逐层收缩能力。第一层是Tool Profile，只向当前Runtime模式暴露必要工具；第二层是Schema与路径解析，拦截非法参数、`../`、绝对路径和符号链接逃逸；第三层是Permission，根据只读模式、Plan Mode、HITL审批和Worker write scope处理高风险命令与越权写入；第四层是Tool Policy，执行修改前fresh read、专用工具优先和重复调用拦截；最后才进入Sandbox、命令匹配和超时控制。每次allow或deny都会进入治理事件与Trace。Prompt可以告诉模型规则，但模型可能忽略或被注入攻击影响，所以真正的路径、权限和副作用限制必须在模型外执行。”

#### Q35：S45为什么失败，意味着什么，如何修复？

- **源码文件**：`pico/features/memory.py`、`memory_quarantine.py`、`evaluation/metrics.py`。
- **答题骨架**：拒绝事件存在 → 最终存储检查失败 → 原子一致性 → 全路径回归。
- **🎙️ 60-120 秒示范稿**：
  > “S45是50个真人场景中唯一失败项，测试API Key形态内容是否会进入Durable Memory。运行过程中已经记录`dependency-facts:secret_shaped`拒绝事件，说明前置检测识别到了风险；但最终`secret_not_promoted`检查失败，说明某条持久化路径仍然留下了内容。根因不是模型没有收到安全Prompt，而是拒绝决策和最终写入没有收口到同一个原子入口。修复时我会把Secret检测放到统一Durable promotion函数，在写主题文件、索引和元数据之前一次判定；然后清理既有副本，使相关摘要失效，并扫描Trace与Report。验证顺序是先加单元测试，再单跑S45，最后重跑完整50场景，确保修复没有误伤正常记忆沉淀。”

#### Q36：“错误接受率0%”具体指什么？敏感信息已经写入如何补救？

- **源码文件**：`pico/features/memory.py`、`memory_quarantine.py`、`pico/core/run_store.py`。
- **答题骨架**：恢复false accept定义 → 不是总安全指标 → 隔离、删除、失效、审计、回归。
- **🎙️ 60-120 秒示范稿**：
  > “这里的错误接受率0%是恢复安全指标，指Schema不兼容、Workspace漂移或恢复证据缺失时，系统没有把旧状态错误地标记为可直接继续。它不能被解释成整个系统所有安全错误都是0%，因为S45属于另一类持久化一致性问题。若敏感信息已经进入Durable Memory，补救也不能只删除界面里看到的一行文本：首先隔离相关候选，阻止继续召回；然后清理主题文件、索引和元数据中的副本，使依赖它的摘要与缓存失效；接着扫描Trace、Report和Session工件是否泄漏，并留下审计记录；最后用Secret Exposure、S45和正常记忆写入Case做正反回归。安全治理必须覆盖拒绝、存储、检索和清理整个生命周期。”

---

### 📍 模块六：评测体系与指标可信度

#### 🧩 实现总览：如何分层验证Agent机制和真实模型效果

Agent评测不能只看最终回答，因为同一个答案可能来自正确执行，也可能来自模型猜测；同样，机制单测通过也不代表真实模型愿意按预期使用。我因此把评测拆成四层：Pytest验证局部代码合同；确定性Harness Benchmark固定模型输出，验证状态机、工具、恢复和门禁；消融实验通过开关单一机制验证因果方向；DeepSeek Live矩阵与真人场景检验真实模型行为和端到端产物。

每个任务都尽量定义确定性Verifier，例如文件内容、pytest结果、Report字段和Trace事件；LLM Judge只用于难以规则化的语义维度。不同变体使用隔离Workspace，固定模型、温度、步骤预算和Fixture，并保留完整Run Evidence。API超时、凭据或Endpoint错误单独标记为Infrastructure Failure，只有模型有效运行后未满足预期，才计入Quality Failure。

```text
Layer 1  Pytest
         Parser / State / Permission / Storage Contract
                         |
                         v
Layer 2  Deterministic Harness Benchmark
         Scripted Model + File/Report/Trace Verifier
                         |
                         v
Layer 3  Ablation & DeepSeek Live Matrix
         Feature On/Off + Repetitions + Usage
                         |
                         v
Layer 4  Human Scenarios / Gate8 / Dogfood
         End-to-End Artifact Acceptance
```

| 评测层 | 回答的问题 | 当前代表产物 |
| :--- | :--- | :--- |
| 单元测试 | 单个模块是否正确 | 513 passed |
| Harness Benchmark | 确定性合同是否稳定 | 12任务三项100% |
| 消融与Live矩阵 | 机制是否产生真实效果 | Context、Memory、Recovery、Handoff |
| 场景验收 | 多模块组合能否完成任务 | Human50、Gate8、Dogfood |

> 🎯 **这一模块的面试主线**：先区分代码正确、机制稳定、模型效果和场景可用，再为每个指标给出分母、对照与失败案例。

#### Q37：为什么Agent不能只用单元测试？确定性评测和真实模型评测有什么区别？

- **源码文件**：`pico/evaluation/harnessbench.py`、`evaluator.py`、`metrics.py`。
- **答题骨架**：局部合同 → Harness组合行为 → Live模型行为 → 真实场景产物。
- **🎙️ 60-120 秒示范稿**：
  > “Agent不能只靠单元测试，因为单测主要验证Parser、权限判断和状态转换等局部合同，而真实失败往往发生在模型、工具、Workspace和多轮状态组合之后。例如记忆召回函数单测通过，不代表真实模型拿到记忆后就不会重复读文件。Pico因此采用分层评测：第一层是Pytest，验证模块和边界；第二层是确定性Harness Benchmark，用ScriptedModel固定输出，在CI里稳定复现工具、恢复和门禁合同；第三层是DeepSeek Live矩阵，保留真实模型和Provider行为；最后用50个场景检查多模块组合后的文件、状态和Trace产物。四层分别回答代码是否正确、机制是否稳定、真实模型是否配合，以及端到端任务是否完成，任何单层100%都不能替代其他层。”

#### Q38：12个Harness Benchmark具体覆盖什么？

- **源码文件**：`pico/evaluation/evaluator.py`、`harnessbench.py`、`benchmarks/coding_tasks.json`。
- **答题骨架**：五类任务 → 确定性Verifier → 能证明与不能证明。
- **🎙️ 60-120 秒示范稿**：
  > “12个Harness Benchmark不是12道随机编码题，而是专门验证运行时合同。它包括2个文档修改、2个文本修改、3个工具边界恢复、3个Checkpoint与Freshness恢复、2个Durable Memory写入合同。工具边界覆盖非法Patch参数、路径逃逸和重复读取；恢复部分覆盖上下文缩减Checkpoint、文件新鲜度重锚和Workspace mismatch；记忆部分分别检查允许沉淀和拒绝Secret或临时状态。每个任务都有确定性Verifier，直接检查文件内容、Report字段和Trace事件。最终任务通过率、Verifier通过率和预算内完成率都是100%。它能证明这些Harness合同没有被重构破坏，但不能代表真实模型开放式编码能力或生产用户成功率。”

#### Q39：Verifier如何判断成功，为什么不能只用LLM Judge？

- **源码文件**：`pico/evaluation/evaluator.py`、`pico/core/verification.py`。
- **答题骨架**：确定性事实优先 → Judge只补语义 → 分离Judge失败与Agent失败。
- **🎙️ 60-120 秒示范稿**：
  > “Verifier的原则是能用确定性事实判断的，就不交给另一个模型。文件是否存在、指定文本是否修改、pytest是否通过、工具是否越权、Checkpoint状态是否正确，都可以通过文件断言、命令退出码、Report字段和Trace事件稳定验证。LLM Judge更适合语义等价、解释完整性或无法写成规则的开放答案。如果所有结果都交给Judge，会引入温度、Prompt偏差和模型版本变化，甚至把Judge错误误算成Agent失败。Pico因此先用确定性Verifier检查执行事实，必要时再增加Judge作为补充维度，并单独记录Judge unavailable或infrastructure error。这样同一个Run可以复算，也能明确失败发生在Agent、Verifier还是Judge。”

#### Q40：消融实验如何控制变量？

- **源码文件**：`pico/evaluation/metrics.py`、`context_cost.py`、`memory_agent_eval.py`。
- **答题骨架**：固定任务/环境/模型/预算/Verifier → 只切一个机制 → 隔离Workspace → 配对聚合。
- **🎙️ 60-120 秒示范稿**：
  > “消融实验的核心是只改变一个机制，其余条件尽量一致。Pico会固定Task Fixture、Workspace初始状态、Provider、模型、温度、步骤预算和Verifier，再切换目标Feature。上下文实验比较Full与Raw；记忆比较memory on、off和irrelevant；恢复比较resume enabled与disabled；Handoff则在同一长会话任务上配对比较Compact和LLM Handoff。每个变体使用独立临时Workspace，避免前一个Run留下的文件或Memory污染后一个Run；同一任务重复运行并保留Trace，聚合时按Task ID配对，而不是把不同难度任务直接平均。这样观察到的差异才更可能来自被切换的机制。即便如此，真实模型仍有随机性，所以结果还要报告重复次数、失败数和质量退化，不能只挑最好一轮。”

#### Q41：50个真实场景如何设计，98%意味着什么？

- **证据文件**：`human50/summary.json`、各场景`report.json`与`trace.jsonl`。
- **答题骨架**：覆盖范围 → 预期产物/状态 → 49/50 → 失败边界。
- **🎙️ 60-120 秒示范稿**：
  > “这50个场景不是把单元测试换个名字，而是按用户实际使用方式驱动CLI、TUI、Skills、子任务、Plan/Todo、任务恢复、上下文压力、Provider错误和安全边界。每个场景预先定义可验证结果，例如目标文件和测试是否存在、Session是否续接、Todo是否完成、权限或安全事件是否进入Trace。最终49个通过，唯一失败是S45的Secret-shaped记忆拒绝一致性，因此通过率是98%。当前Summary没有保存统一的能力分类字段，所以我只能按场景定义说明覆盖范围，不会编造分类数量。S45失败后仍写98%，是因为这个数字本身已经包含失败：分母50、通过49、失败Case和原因都公开。评测的价值不是制造全绿，而是给出可复现的能力边界。”

#### Q42：Live实验如何控制随机性，重复了几次，如何区分基础设施失败？

- **源码文件**：`pico/evaluation/metrics.py`、`run_evidence.py`、`scripts/run_llm_handoff_benchmark.py`。
- **答题骨架**：固定配置 → 隔离Workspace → 多次重复 → 分层归因。
- **🎙️ 60-120 秒示范稿**：
  > “Live实验先固定DeepSeek V4 Flash、温度、Fixture、功能开关、步骤预算和Verifier，并让每个Run在隔离Workspace执行。上下文每种配置重复5次、两个变体共120次调用；真实记忆每个任务重复5次，三个变体汇总360次调用；安全和长会话Handoff各重复3次。5次主要用于观察不同配置的稳定平均，成本更高、链路更长的实验取3次，是在波动、运行时间和API费用之间折中，这些重复数用于发现方向，不用于宣称统计显著性。归因时，凭据错误、Endpoint不可用、API超时和评测器异常单独标记为infrastructure error，不进入有效产品指标；只有模型完成运行但文件、状态或Verifier不符合预期，才记为quality failure。所有有效Run保留usage、Report和Trace，避免只汇报最好一次。”

#### Q43：当前评测体系最大的缺陷是什么？如果只能补一类评测会补什么？

- **证据文件**：`SUMMARY.json`、`REPORT.md`、`deepseek-v4-flash-real-metrics.json`。
- **答题骨架**：受控Fixture偏多 → 产物Schema缺口 → 真实仓库长链路配对回归。
- **🎙️ 60-120 秒示范稿**：
  > “我认为当前最大缺陷不是缺少更多百分比，而是评测分布仍以受控Fixture和短周期任务为主，对真实仓库、跨天恢复、数十步长链路和不同模型的覆盖不足；同时产物Schema还不完全统一，例如Prefix Cache独立实验没有并入最新全量报告，Memory Challenge没有单独落盘固定K，安全矩阵也缺少统一的`expected/actual/passed`字段。如果只能新增一类评测，我会优先做真实仓库长链路配对回归：固定同一个仓库起点，让基线与Pico机制多次完成跨文件修改、测试失败修复和中断恢复。指标同时看最终Verifier、恢复后的首动作、重复工具调用、Provider总Token、墙钟时间和安全事件，并保留失败Trace。这样最能验证多个机制组合后是否真正提升工程任务，而不只是单项指标好看。”

---

## 五、 最新实验数据与指标防守

### 5.1 📊 全量评测概览

| 评测层 | 规模 | 核心结果 | 面试中能证明什么 |
| :--- | :--- | :--- | :--- |
| Pytest | 513 passed，2 skipped | 0 failed | 基础模块回归 |
| Harness Benchmark | 12任务 | 通过、Verifier、预算内完成均100% | 确定性Harness合同 |
| 真实上下文矩阵 | 120次Live调用 | 平均压缩9.48%，正确率100% | 当前矩阵的字符压缩与保真 |
| Memory Challenge | 55案例 | Accuracy94.55%，Recall100%，Precision87.23% | 记忆召回、更新和安全质量 |
| 任务恢复 | 开启/关闭各33次 | 恢复90%，漂移100%，false accept0% | 受控恢复机制收益 |
| 真实安全矩阵 | 11场景×3 | Path Escape事件9次；工具错误码完整记录 | 模型请求可进入Runtime拒绝路径 |
| 真人场景 | 50场景 | 49/50，98% | 多模块端到端验收 |
| Gate8 | 9场景 | 9/9，Live Smoke通过 | 核心发布门禁 |

### 5.2 📉 上下文实验

| 指标 | Raw / 对照 | Full / 治理后 |
| :--- | ---: | ---: |
| 平均Prompt字符 | 13,474 | 12,024.92 |
| 平均压缩率 | — | 9.48% |
| 最高 / 最低压缩率 | — | 19.57% / -2.53% |
| 正确率 | 100% | 100% |

> 🛡️ **面试防守要点**：9.48%不能说成Token或账单成本下降；85.3%是独立Prefix Cache实验。

### 5.3 🧠 Memory Challenge

| 指标 | Memory On |
| :--- | ---: |
| 案例数 | 55 |
| Answer Accuracy | 94.55% |
| Evidence Recall@K | 100% |
| Evidence Precision@K | 87.23% |
| Stale Use | 0% |
| Secret Exposure | 0% |
| Failed Cases | 3 |

> 🛡️ **面试防守要点**：三个失败都是冲突证据使用错误，不是检索漏召回。

### 5.4 🔬 真实记忆矩阵

| 指标 | Memory On | Memory Off | Irrelevant Memory |
| :--- | ---: | ---: | ---: |
| 正确率 | 100% | 100% | 100% |
| 重复读取 | 67 | 58 | 69 |
| 平均模型尝试 | 2.48 | 2.10 | 2.40 |
| 平均工具步数 | 1.12 | 0.97 | 1.15 |

> 🛡️ **面试防守要点**：真实模型下没有形成读取或成本收益，简历不写“真实模型重复读取下降”。

### 5.5 🔁 LLM Handoff 配对实验

| 指标 | Compact | LLM Handoff |
| :--- | ---: | ---: |
| 成功率 | 86.67% | 86.67% |
| Verifier通过率 | 93.33% | 100% |
| 平均总输入Token | 24,232.87 | 25,265.80 |
| 平均工具步数 | 12.20 | 13.07 |

- 配置价格变化中位数：-5.93%。
- 未缓存输入变化中位数：-15.30%。
- 平均总输入Token变化：+4.26%。
- 质量退化配对：6。
- `claimable_cost_win=false`。

> 🛡️ **面试防守要点**：Handoff有Verifier提升，但当前不能宣称稳定降本。

---

## 六、 失败案例与 Troubleshooting

### 6.1 🐞 案例一：S45 Secret 拒绝事件与最终存储状态不一致

- **现象**：Trace记录`dependency-facts:secret_shaped`拒绝，但`secret_not_promoted`检查失败。
- **定位路径**：拒绝事件 → Durable promotion入口 → 主题文件 → 索引与元数据 → 最终Verifier。
- **根因判断**：安全决策与持久化路径没有收口到同一个原子入口。
- **修复方案**：在统一写入口执行敏感检测；拒绝后禁止写入文件、索引和元数据；清理既有副本并使摘要失效。
- **回归方式**：单跑S45 → Secret Exposure回归 → 完整重跑50场景。

**🎙️ 面试话术**：

> “我不会把失败场景藏起来。S45说明‘记录了拒绝’不等于‘最终一定没写入’，暴露的是安全判定和持久化的一致性问题。它让我把修复目标从增加一条Prompt规则，提升为统一写入口和全存储位置Verifier。”

### 6.2 🧠 案例二：真实模型开启记忆后重复读取增加

- **现象**：Memory On重复读取67次，Memory Off为58次。
- **离线对照**：ScriptedModel中重复读取60降至0。
- **根因判断**：记忆成功召回，但真实模型不稳定信任已注入证据，产生验证性读取。
- **优化方向**：记忆条目显式携带来源、文件版本、时间和可信状态；只在证据缺失、冲突或过期时建议重新读取。
- **验证指标**：正确率保持100%，同时Memory On读取、工具步数和总Token低于Off。

### 6.3 📉 案例三：短上下文出现 -2.53% 压缩收益

- **现象**：治理后Prompt比Raw略长。
- **根因判断**：Section标题、摘要引用和编排元数据有固定开销，短历史没有足够冗余可回收。
- **优化方向**：估算可回收字符与治理固定成本，只有预计净收益为正时才启用压缩。
- **验证方式**：按短、中、长上下文分桶报告，不只看全局平均值。

### 6.4 🔁 案例四：Handoff 局部输入下降但总输入增加

- **现象**：未缓存输入中位数下降15.30%，平均总输入Token增加4.26%。
- **根因判断**：Handoff增加一次摘要调用，并可能引发更多工具步骤和模型尝试。
- **优化方向**：建立净收益预测，只在历史足够长、摘要复用轮次足够多且质量门禁通过时启用。
- **验证方式**：同时报告Provider usage、工具步数、成功率、Verifier和质量退化，不单看配置价格。

---

## 七、 评分标准与 3 天训练计划

### 7.1 🧭 三阶评分标准

| 维度 | 危险答案 | 合格答案 | 优秀答案 |
| :--- | :--- | :--- | :--- |
| 项目定位 | “就是一个会调用工具的Agent” | 能说明本地Coding场景 | 清楚区分模型决策与Harness治理 |
| 执行链路 | 只背模块名 | 能说出模型—工具循环 | 能讲状态、权限、证据和终态转换 |
| 上下文 | 把字符当Token或成本 | 能解释稳定前缀与压缩 | 能给阈值、样本、对照和负收益 |
| 记忆 | 只说“用了RAG” | 能讲分层与召回 | 能解释Recall/Accuracy差异和3个失败 |
| 恢复安全 | “保存聊天记录再加载” | 能讲Checkpoint和指纹 | 能解释30/33、no-checkpoint与S45 |
| 评测 | 只报100% | 能讲Benchmark和场景 | 能区分单测、消融、Live、基础设施失败 |

### 7.2 ✅ 面试前必须能脱稿回答的十题

1. Pico与普通ReAct的区别是什么？
2. 一次请求如何从Prompt走到工具，再走到Final？
3. 为什么完成判定必须放在模型外？
4. 85.3%和9.48%分别是什么实验？
5. 四级压力阈值与裁剪顺序是什么？
6. Recall@K 100%为什么Accuracy只有94.55%？
7. 为什么真实模型下记忆反而增加重复读取？
8. 恢复成功率90%的分母和失败项是什么？
9. S45暴露了什么一致性问题？
10. 12任务100%与50场景98%分别能证明什么？

### 7.3 📅 三天练习计划

**Day 1：建立项目主线**

- 练熟1分钟和3分钟开口稿。
- 不看文档画出Runtime全链路。
- 用一句话区分模型、Agent与Harness。

**Day 2：源码与指标防守**

- 沿`engine.py → model_output.py → tool_executor.py → final_readiness.py`读执行链。
- 沿`context_pressure.py → context_manager.py → context_handoff.py`读上下文链。
- 沿`memory.py → runtime_checkpoints.py → evaluation/metrics.py`读记忆、恢复和评测链。
- 对85.3%、9.48%、94.55%、90%和98%分别说出样本、分母、单位和边界。

**Day 3：连续追问与失败复盘**

- 围绕上下文、记忆和S45各练一条三轮追问链。
- 每个回答控制在60到120秒，前15秒先给结论。
- 主动讲一个被实验推翻的假设：真实模型下记忆没有减少重复读取。
- 对无法从产物确认的数字明确说“当前报告未落盘”，不要临场补造。

### 7.4 🗒️ 面试当天速记卡

```text
一句话定位：
Pico让模型负责提出下一步，让Harness负责权限、状态、证据和完成判定。

五个核心指标：
Prefix Cache 85.3%：独立缓存实验；
Prompt 9.48%：120次DeepSeek Live，字符压缩，正确率100%；
Memory 94.55%：55个Challenge，Recall@K 100%；
Recovery 90%：11任务×3，30/33，Drift 100%，False Accept 0%；
Scenario 98%：49/50，失败S45。

三个不能混淆：
字符压缩不等于Token或成本下降；
Recall不等于最终答案准确率；
确定性Benchmark不等于真实模型或生产表现。

两个主动承认的不足：
真实模型下记忆没有减少重复读取；
S45存在拒绝决策与最终持久化状态的一致性缺口。
```

### 7.5 🔍 本轮追问覆盖核对

| 追问主线 | 对应题目 | 覆盖状态 |
| :--- | :--- | :--- |
| 项目定位、框架选型、ReAct区别、个人贡献、长链路、项目阶段 | Q1-Q6 | 已覆盖并提供示范答案 |
| Runtime流程、状态转换、工具解析、异常处理、完成门禁、预算与Trace | Q7-Q13 | 已覆盖并提供示范答案 |
| 稳定前缀、缓存对照、压力等级、裁剪顺序、9.48%实验、负收益、Handoff | Q14-Q21 | 已覆盖并提供示范答案 |
| 分层记忆、自动沉淀、冲突与新鲜度、检索、55案例、Recall@K、3个失败、真实模型效果 | Q22-Q29 | 已覆盖并提供示范答案 |
| Checkpoint、恢复下一步、漂移、90%分母、错误接受、工具权限、S45与补救 | Q30-Q36 | 已覆盖并提供示范答案 |
| 分层评测、12任务、Verifier、消融、50场景、Live重复与基础设施归因、评测缺口 | Q37-Q43 | 已覆盖并提供示范答案 |

> ⚠️ **仍需补强的是原始证据，不是问答缺失：**

1. 前缀缓存独立实验已有12.1%与85.3%的文档记录，但缺少最新全量重跑和可复算Provider明细。
2. Memory Challenge保存了Ground Truth与召回结果，但没有把固定K单独写入汇总Schema。
