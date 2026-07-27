# Pico 技术文档与面试指南导航地图

欢迎来到 `pico` 项目技术文档库。本目录包含 Pico 项目的架构白皮书、单向阅读链路、记忆系统、评测体系、面试话术与源码技术内幕。

---

## 📁 磁盘真实中文文件目录结构与直接导航

- **架构与技术设计白皮书 (单向终极阅读链路 📖)/**
  - [00_架构与技术设计白皮书全景导航.md](./架构与技术设计/00_架构与技术设计白皮书全景导航.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/00_%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1%E7%99%BD%E7%9A%AE%E4%B9%A6%E5%85%A8%E6%99%AF%E5%AF%BC%E8%88%AA.md)) — 【白皮书全景导航】单向终极阅读链路入口与全章索引
  - [01_设计哲学与架构选型(ADR).md](./架构与技术设计/01_设计哲学与架构选型(ADR).md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/01_%E8%AE%BE%E8%AE%A1%E5%93%B2%E5%AD%A6%E4%B8%8E%E6%9E%B6%E6%9E%84%E9%80%89%E5%9E%8B%28ADR%29.md)) — Chapter 1: 痛点分析、四大工程解法与 7 大 ADR 架构选型推导
  - [02_总体架构与两条主线生命周期.md](./架构与技术设计/02_总体架构与两条主线生命周期.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/02_%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84%E4%B8%8E%E4%B8%A4%E6%9D%A1%E4%B8%BB%E7%BA%BF%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.md)) — Chapter 2: 系统分层图谱、装配线/执行线与 10 阶段运行生命周期
  - [03_受控工具网关与安全质量门禁.md](./架构与技术设计/03_受控工具网关与安全质量门禁.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/03_%E5%8F%97%E6%8E%A7%E5%B7%A5%E5%85%B7%E7%BD%91%E5%85%B3%E4%B8%8E%E5%AE%89%E5%85%A8%E8%B4%A8%E9%87%8F%E9%97%A8%E7%A6%81.md)) — Chapter 3: `run_tool` 8 重关卡校验、Read Freshness 与 FinalReadiness 改代码必测试打回闭环
  - [04_上下文编排与前缀缓存优化.md](./架构与技术设计/04_上下文编排与前缀缓存优化.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/04_%E4%B8%8A%E4%B8%8B%E6%96%87%E7%BC%96%E6%8E%92%E4%B8%8E%E5%89%8D%E7%BC%80%E7%BC%93%E5%AD%98%E4%BC%98%E5%8C%96.md)) — Chapter 4: ContextOrchestrator 编排、85%+ Prompt Caching 命中与 ContextPressure 梯队计算
  - [05_分层记忆系统与数据检疫治理.md](./架构与技术设计/05_分层记忆系统与数据检疫治理.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/05_%E5%88%86%E5%B1%82%E8%AE%B0%E5%BF%86%E7%B3%BB%E7%BB%9F%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%A3%80%E7%96%AB%E6%B2%BB%E7%90%86.md)) — Chapter 5: 四层 Markdown 记忆结构、(tag, keyword, recency) 算分召回与 Quarantine 隔离池
  - [06_任务规划与多Agent隔离协同.md](./架构与技术设计/06_任务规划与多Agent隔离协同.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/06_%E4%BB%BB%E5%8A%A1%E8%A7%84%E5%88%92%E4%B8%8E%E5%A4%9AAgent%E9%9A%94%E7%A6%BB%E5%8D%8F%E5%90%8C.md)) — Chapter 6: PlanMode 规则模式、TodoLedger 状态机流转与 Worker write_scope 路径隔离
  - [07_审计工件落盘与确定性回放评测.md](./架构与技术设计/07_审计工件落盘与确定性回放评测.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/07_%E5%AE%A1%E8%AE%A1%E5%B7%A5%E4%BB%B6%E8%90%BD%E7%9B%98%E4%B8%8E%E7%A1%AE%E5%AE%9A%E6%80%A7%E5%9B%9E%E6%94%BE%E8%AF%84%E6%B5%8B.md)) — Chapter 7: 3 层评测体系、trace.jsonl 协议、FakeModelClient 确定性回放与三大消融实验
  - [08_二次开发指南与故障诊断手册.md](./架构与技术设计/08_二次开发指南与故障诊断手册.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/08_%E4%BA%8C%E6%AC%A1%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97%E4%B8%8E%E6%95%85%E9%9A%9C%E8%AF%8A%E6%96%AD%E6%89%8B%E5%86%8C.md)) — Chapter 8: Custom Tool/Provider/Skill 扩展开发 SOP 与 4 大高频故障 Troubleshooting
  - [09_项目演进与main分支代码对比.md](./架构与技术设计/09_项目演进与main分支代码对比.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AE%BE%E8%AE%A1/09_%E9%A1%B9%E7%9B%AE%E6%BC%94%E8%BF%9B%E4%B8%8Emain%E5%88%86%E6%94%AF%E4%BB%A3%E7%A0%81%E5%AF%B9%E6%AF%94.md)) — Chapter 9: 从 main 分支到 v3 的 35,000+ 行重构对比与 5 大核心模块代码级 Diff

- **面试高频实战 (大一统唯一终极指南 ⭐️)/**
  - [Pico_v3_AI_Agent工程师面试全景终极指南.md](./面试高频实战/Pico_v3_AI_Agent工程师面试全景终极指南.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E9%9D%A2%E8%AF%95%E9%AB%98%E9%A2%91%E5%AE%9E%E6%88%98/Pico_v3_AI_Agent%E5%B7%A5%E7%A8%8B%E5%B8%88%E9%9D%A2%E8%AF%95%E5%85%A8%E6%99%AF%E7%BB%88%E6%9E%81%E6%8C%87%E5%8D%97.md)) — **【唯一必备】**全量简历原文本、自我介绍、演进重构、ADR 选型辩论、15 大全景问点示范稿、源码 Hook、Troubleshooting 踩坑案例、三阶评分表与 3 天计划无损合一终极指南！

- **官方参考手册/**
  - [configuration.md](./官方参考手册/configuration.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/configuration.md)) — 配置合并优先级与 .pico.toml 说明
  - [memory.md](./官方参考手册/memory.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/memory.md)) — 记忆功能使用与分层结构说明
  - [sandbox.md](./官方参考手册/sandbox.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/sandbox.md)) — 命令防护沙箱与安全隔离说明
  - [skills.md](./官方参考手册/skills.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/skills.md)) — Skills 技能扩展机制与结构化 SOP 说明

---

## 🗺️ 面试与复习推荐路线

1. **白皮书全景** ➡️ 从 [00_架构与技术设计白皮书全景导航.md](./架构与技术设计/00_架构与技术设计白皮书全景导航.md) 进入，顺着 **Chapter 1 至 Chapter 9** 研读底层原理。
2. **面试大一统指南 (只看这一个文档即可 🎯)** ➡️ **查阅 [Pico_v3_AI_Agent工程师面试全景终极指南.md](./面试高频实战/Pico_v3_AI_Agent工程师面试全景终极指南.md)**！包含了自我介绍、简历原文本、全量 15 个项目问点、源码 Hook、Troubleshooting 案例与逐字示范稿！
