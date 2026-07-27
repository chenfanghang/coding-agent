# Pico v3 架构与技术设计白皮书

> 📖 **【单向阅读链路 - Chapter 1/9】**  
> **[下一章：02_总体架构与两条主线生命周期 ➡️](./02_总体架构与两条主线生命周期.md)**

---

# Chapter 1: 设计哲学与架构选型 (ADR)

欢迎阅读 `pico v3` 架构与技术设计白皮书。本文作为整条阅读链路的起始章节，重点解答：**Pico 的工程定位是什么？在本地 Coding 场景下面临哪些核心工程矛盾？基于第一性原理与严谨工程推导，为什么做出三大核心 ADR 选型决策？**

---

## 1.1 系统概述与工程定位

`pico v3` 是一个运行在本地 Git 代码仓库中的轻量级 **Local Coding Agent Harness**。它并非简单的聊天终端，而是一个具备工作区指纹感知、受控工具网关、完成治理门禁 (Final Readiness)、分层记忆检疫 (Quarantine) 和运行审计工件的自动编程助手。

### 核心工程使命：
- **开发过程受控化**：通过 ToolProfile 限制工具暴露面，防止 Shell 危险命令与未经验证的死循环。
- **状态恢复确定化**：通过原子状态镜像与 Git Commit SHA/Hash 漂移校验，支持高可靠的会话断点恢复。
- **上下文成本最优化**：通过 `ContextOrchestrator` 编排静态 Prefix，实现大模型前缀缓存 (Prompt Caching) 的稳定高命中。
- **验证就绪硬门禁**：通过 `FinalReadiness` 治理逻辑，解决 Agent 改完代码不验证就急于答复的盲目退出痛点。

---

## 1.2 四大工程矛盾与 Pico v3 核心解法

在设计 Pico v3 时，我们针对通用 Agent 框架在本地 Coding 场景下的四大核心工程矛盾进行了解法匹配：

| 工程矛盾 | Pico v3 核心架构解法 | 对应源码实现模块 | 带来的工程收益 |
| :--- | :--- | :--- | :--- |
| **1. 上下文爆炸与 Token 高昂** | ContextOrchestrator 动态编排 + 静态 Prefix | [context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_orchestrator.py) | 前缀缓存高命中率，显著降低推理延迟与成本 |
| **2. 盲目提早退出 (Premature Exit)** | FinalReadiness 完成治理硬性验证门禁 | [final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py) | 解决改完代码不测试就答复的通病，未验证退出大幅下降 |
| **3. 坏记忆与幻觉污染数据库** | MemoryQuarantine 检疫隔离池 + Lint 校验 | [memory_quarantine.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory_quarantine.py) | 自动拦截矛盾与冲突数据，避免数据库污染 |
| **4. 高危 Shell 命令与盲改代码** | Read Freshness Guard + Shell 隔离沙箱 | [tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py) & [sandbox](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/sandbox) | 强制改代码前先 read；阻断危险系统级 Shell 命令 |

---

## 1.3 核心架构决策记录 (ADR 专集)

### ADR-001: 纯 Python 轻量化自研 vs 开源 Agent 框架 (如 LangChain/LangGraph) 选型

- **背景**：在项目初期，需评估是基于已有的开源 Agent 框架搭建，还是采用纯 Python 自研 Harness。
- **第一性原理推导与严谨对比 (Trade-offs)**：
  1. **Prompt 字节级控制与前缀缓存契约**：大模型 Provider（如 DeepSeek/Anthropic）的前缀缓存要求从 Prompt 第 1 个字节开始的 **Prefix 必须严格保持静态匹配**。开源框架为了支持通用扩展，在 Prompt 渲染层往往注入了框架级变量、动态插件描述或运行上下文，一旦这些动态字节散落在头部，会导致整个静态 Prefix 的哈希改变而触发 Cache Miss。自研 Harness 的 `ContextOrchestrator` 能对 Prompt 的字节布局拥有绝对掌控力，保证 Header 的绝对静态。
  2. **完成治理契约的内聚度**：虽然开源框架可以通过中间件 (Middleware) 或自定义 Output Parser 拦截输出，但通用框架的图节点（StateGraph）多将“无 tool_call 的输出”直接判定为 End 结束节点。要结合跨 Step 的 `has_code_changes` 物理改动证据与 `pytest` 运行证据进行打回，在通用框架中需要重构 State 模式或编写深度侵入的回调；而 Pico 自研 Harness 将 `FinalReadiness` 作为 Engine 调度的**一等公民 (First-class Citizen)**，拦截打回与状态机原生内聚。
  3. **启动耗时与依赖开销**：通用框架包含较深的抽象与较多的第三方依赖，初始化耗时在秒级；Pico 零重型框架依赖，启动耗时 **< 50ms**，内存占用不到 30MB，极度适合命令行和 IDE 内置终端使用。
- **决策结论**：**选择纯 Python 轻量化自研 Harness**。

---

### ADR-002: 上下文编排设计哲学与 Prompt Caching 静态放置 ADR

- **背景**：随着会话轮数增加，Prompt 长度剧增。若每次组装 Prompt 的 Prefix 发生抖动，会导致大模型 API 的前缀缓存失效。
- **第一性原理推导 (Trade-offs)**：
  - **设计哲学**：在 [pico/core/context_orchestrator.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_orchestrator.py) 中实施“静态与动态强分隔”。
  - **静态 Prefix 置前**：将绝对固定不变的 System Prompt、Tool 签名强制置于 Prompt 最前端，计算生成固定的 `prefix_hash`，死死锁定缓存。
  - **动态 Section 置后**：将变动的 Memory 召回、Workspace 快照与对话历史置于尾部。
  - **自适应压缩**：在 [pico/core/context_pressure.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/context_pressure.py) 中计算 `pressure_ratio`。当达到 `tier3_summary` (95%) 且历史 > 4 轮时，自动触发 `compact_history()` 释放 Token 占用。
- **决策结论**：**采用强分隔 ContextOrchestrator + 静态 Prefix 锁定**。在实验组对比测试中显著提升前缀缓存复用率。

---

### ADR-003: 记忆系统设计哲学与分层 Markdown + Quarantine 检疫池 ADR

- **背景**：需决定记忆系统是采用 Embedding 向量数据库 (Vector DB/RAG) 还是明文文件系统。
- **第一性原理推导 (Trade-offs)**：
  1. **精密代码匹配 vs 语义模糊检索**：向量语义相似度在代码变量名、正则与配置项上存在离散误差（如 `jwt_secret` 与 `api_secret` 向量距离很近但属于完全不同的配置），容易召回无关代码噪声；而在代码场景下，基于 Tag 与关键词标量匹配能实现 100% 精确召回。
  2. **文件变动成本**：代码修改极频繁，若采用 RAG 重新切片 Embedding，CPU 与 API 消耗不可接受；而分层 Markdown 文件更新元数据摘要无延迟。
  3. **数据治理与幻觉隔离**：模型产生的错乱回答容易写坏长期库。在 [pico/features/memory_quarantine.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/features/memory_quarantine.py) 中引入 `MemoryQuarantine`，将有矛盾的候选记忆拦截入 `quarantine/` 隔离池，结合后台 PID 锁安全归纳。
- **决策结论**：**放弃 Vector DB，采用四层明文 Markdown + `(tag, keyword, recency)` 三元组算分召回 + MemoryQuarantine 检疫池**。

---

### ADR-004: 受控工具网关设计哲学与 Read Freshness + 沙箱防护 ADR

- **背景**：如何防止 Agent 盲改文件、死循环调用工具以及运行高危 Shell 命令（如 `rm -rf`）破坏本地系统。
- **第一性原理推导 (Trade-offs)**：
  1. **防盲改文件 (Read Freshness Guard)**：在 `ToolPolicyChecker` 中强校验：未经读取 (`read_file`) 的物理路径，禁止发起 `patch_file` 或 `write_file`，从根源消除覆盖未知代码的隐患。
  2. **8 重关卡治理网关**：在 [pico/core/tool_executor.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/tool_executor.py) 中挂载 Pydantic v2 参数强类型校验、路径逃逸防御 (`path_escape`)、重复调用防卡死拦截 (`repeated_tool_call`) 以及物理快照 Diff 比对。
  3. **Shell 安全沙箱**：隔离高危 Shell 指令，敏感命令触发 TUI HITL (Human-in-the-Loop) 审批。
- **决策结论**：**建立 Pydantic v2 + Read Freshness Guard + 8 重治理工具网关**。

---

### ADR-005: 完成治理设计哲学与 FinalReadiness 改代码必测试门禁 ADR

- **背景**：如何解决 Agent 改完代码未验证就急于输出 `<final>` 答复给用户的 **Premature Exit (提早盲目退出)** 痛点。
- **第一性原理推导 (Trade-offs)**：
  - 纯依赖 Prompt 中提示“改完请自己跑测试”收效甚微，模型经常跳过测试。
  - 在 [pico/core/final_readiness.py](file:///Users/chenfanghang/PycharmProjects/pico/pico/core/final_readiness.py) 中建立框架调度层硬门禁：当模型试图给最终答复时，拦截进入 `evaluate_final_readiness`，强校验 `has_code_changes` 与 `pytest` 运行证据。未验证则触发 `action == "block"` 强行阻断退出并打回循环。
- **决策结论**：**采用 FinalReadiness 完成治理硬门禁**。

---

### ADR-006: 磁盘明文存储 (`.pico/`) vs 关系型/向量数据库选型 ADR

- **背景**：确定会话快照、记忆与审计日志的落盘存储格式。
- **第一性原理推导 (Trade-offs)**：
  - 二进制数据库（如 SQLite）无法直观与 Git diff 兼容，且调试时需要借助专业客户端。
  - Pico 选择磁盘隐藏目录 `.pico/` 下的明文 JSON/Markdown 存储：**100% 人类可读、可编辑、可干预**，且方便一键清理与提交审计。
- **决策结论**：**采用磁盘明文 JSON/Markdown 存储**。

---

### ADR-007: Textual TUI 终端界面 vs Web UI / Electron 架构选型 ADR

- **背景**：确定 Agent 的用户交互界面形态。
- **第一性原理推导 (Trade-offs)**：
  - Web UI / Electron 需要启动数百兆内存的浏览器环境，且在 Linux 远程 SSH 终端开发时不友好。
  - Pico 选择基于 Python Textual 的响应式 TUI 终端界面：内存占用 **< 30MB**，原生运行于本地 Terminal、IDE 终端及远程 SSH。
- **决策结论**：**采用 Textual TUI 终端交互形态**。

---

> 📖 **【单向阅读链路 - 本章结束】**  
> **[⬅️ 返回白皮书全景导航 🏠](./00_架构与技术设计白皮书全景导航.md) | [下一章：02_总体架构与两条主线生命周期 ➡️](./02_总体架构与两条主线生命周期.md)**
