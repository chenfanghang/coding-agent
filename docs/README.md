# Pico 技术文档与面试指南导航地图

欢迎来到 `pico` 项目技术文档库。本目录包含 Pico 项目的架构白皮书、单向阅读链路、记忆系统、评测体系、面试话术与源码技术内幕。

---

## 📁 磁盘真实中文文件目录结构与直接导航

- **架构与技术设计白皮书（单向阅读链路）/**
  - [00_架构与技术设计白皮书全景导航.md](./架构与技术设计/00_架构与技术设计白皮书全景导航.md) — 全章索引、术语表与推荐阅读路径
  - [01_设计哲学与架构选型(ADR).md](<./架构与技术设计/01_设计哲学与架构选型(ADR).md>) — Chapter 1：系统定位与 6 项核心架构决策
  - [02_总体架构与两条主线生命周期.md](./架构与技术设计/02_总体架构与两条主线生命周期.md) — Chapter 2：装配线、执行线、Turn 状态转换与端到端示例
  - [03_受控工具网关与安全质量门禁.md](./架构与技术设计/03_受控工具网关与安全质量门禁.md) — Chapter 3：工具策略、权限模式、重复调用治理与完成门禁
  - [04_上下文编排与前缀缓存优化.md](./架构与技术设计/04_上下文编排与前缀缓存优化.md) — Chapter 4：稳定前缀、双压力口径、结果落盘与渐进压缩
  - 🧠 **[分层记忆系统 (Memory System)](./记忆/README.md)**：包含 Working Memory、Durable Memory、Quarantine 隔离池、Retrieval 算法与 Auto-Dream 蒸馏。
  - ⚡ **[上下文编排系统 (Context Orchestrator)](./上下文/README.md)**：包含 Prefix Lock 字节强锁、Section Budget 分区预算、4 阶梯剪枝与 Artifact Offloading。
  - 🛠️ **[受控工具网关 (Tool Gateway)](./工具调用/README.md)**：包含 8 重受控关卡、Path Escape 防逃逸、Read Freshness 读后修改保护、Tool Repetition 阻断与 Approval 审批。
  - [05_分层记忆系统与数据检疫治理.md](./架构与技术设计/05_分层记忆系统与数据检疫治理.md) — Chapter 5：分层存储、召回、新鲜度与元数据检疫
  - [06_任务规划与多Agent隔离协同.md](./架构与技术设计/06_任务规划与多Agent隔离协同.md) — Chapter 6：计划状态机、Worker Profile 与写入范围隔离
  - [07_任务恢复与状态一致性.md](./架构与技术设计/07_任务恢复与状态一致性.md) — Chapter 7：Checkpoint、Runtime Identity、恢复判定与漂移处理
  - [08_运行审计与分层评测.md](./架构与技术设计/08_运行审计与分层评测.md) — Chapter 8：Session Event、Run Trace、确定性回放与分层评测
  - [09_二次开发指南与故障诊断手册.md](./架构与技术设计/09_二次开发指南与故障诊断手册.md) — Chapter 9：Tool、Provider、Skill 扩展与故障诊断
  - [10_项目架构演进与能力边界.md](./架构与技术设计/10_项目架构演进与能力边界.md) — Chapter 10：架构演进、当前边界与后续方向

- **面试高频实战 (大一统唯一终极指南 ⭐️)/**
  - [Pico_v3_AI_Agent工程师面试全景终极指南.md](./面试高频实战/Pico_v3_AI_Agent工程师面试全景终极指南.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E9%9D%A2%E8%AF%95%E9%AB%98%E9%A2%91%E5%AE%9E%E6%88%98/Pico_v3_AI_Agent%E5%B7%A5%E7%A8%8B%E5%B8%88%E9%9D%A2%E8%AF%95%E5%85%A8%E6%99%AF%E7%BB%88%E6%9E%81%E6%8C%87%E5%8D%97.md)) — **【唯一必备】**全量简历原文本、自我介绍、演进重构、ADR 选型辩论、15 大全景问点示范稿、源码 Hook、Troubleshooting 踩坑案例、三阶评分表与 3 天计划无损合一终极指南！
  - [Pico完整高频面试题与参考答案.md](./面试高频实战/Pico完整高频面试题与参考答案.md) — 按模块整理实现总览、高频追问与可直接口述的示范稿
  - [Pico实验评测说明-面试版.md](./面试高频实战/Pico实验评测说明-面试版.md) — 面向面试的实验含义、指标口径与结果说明
  - [Pico项目专项面试备书.md](./面试高频实战/Pico项目专项面试备书.md) — 旧版专项材料，保留用于补充查阅

- **评测体系 (设计、口径、复现与结论边界)/**
  - [00_评测体系全景与阅读导航.md](./评测体系/00_评测体系全景与阅读导航.md) — 从评测目标、统一协议进入 Harness、上下文、记忆、恢复、安全与真人场景的完整阅读链路
  - [02_统一实验协议与指标口径.md](./评测体系/02_统一实验协议与指标口径.md) — 对照实验、Verifier、失败分类与指标公式
  - [09_评测产物复现与结果解读.md](./评测体系/09_评测产物复现与结果解读.md) — 运行命令、产物证据链和结果审计清单
  - [10_当前结论局限与后续增强.md](./评测体系/10_当前结论局限与后续增强.md) — 当前结论、反例、局限与增强优先级

- **官方参考手册/**
  - [configuration.md](./官方参考手册/configuration.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/configuration.md)) — 配置合并优先级与 .pico.toml 说明
  - [memory.md](./官方参考手册/memory.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/memory.md)) — 记忆功能使用与分层结构说明
  - [sandbox.md](./官方参考手册/sandbox.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/sandbox.md)) — 命令防护沙箱与安全隔离说明
  - [skills.md](./官方参考手册/skills.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/skills.md)) — Skills 技能扩展机制与结构化 SOP 说明

---

## 🗺️ 面试与复习推荐路线

1. **白皮书全景** ➡️ 从 [00_架构与技术设计白皮书全景导航.md](./架构与技术设计/00_架构与技术设计白皮书全景导航.md) 进入，顺着 **Chapter 1 至 Chapter 10** 研读底层原理。
2. **评测体系** ➡️ 从 [00_评测体系全景与阅读导航.md](./评测体系/00_评测体系全景与阅读导航.md) 进入，系统理解每项实验的意义、对照组、Verifier、数据来源与结论边界。
3. **面试大一统指南 (只看这一个文档即可 🎯)** ➡️ **查阅 [Pico_v3_AI_Agent工程师面试全景终极指南.md](./面试高频实战/Pico_v3_AI_Agent工程师面试全景终极指南.md)**！包含了自我介绍、简历原文本、全量 15 个项目问点、源码 Hook、Troubleshooting 案例与逐字示范稿！
