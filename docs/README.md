# Pico 技术文档与面试指南导航地图

欢迎来到 `pico` 项目技术文档库。本目录包含 Pico 项目的架构说明、记忆系统、评测体系、面试话术与源码技术内幕。

---

## 📁 磁盘真实中文文件目录结构与直接导航

- **架构与技术设计/**
  - [项目架构说明.md](./架构与技术设计/项目架构说明.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AF%B4%E6%98%8E/%E9%A1%B9%E7%9B%AE%E6%9E%B6%E6%9E%84%E8%AF%B4%E6%98%8E.md)) — 核心架构、全路径调用链 (Call Chain)、九大模块与 TUI 异步 UI 架构
  - [记忆实现与面试说明.md](./架构与技术设计/记忆实现与面试说明.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AF%B4%E6%98%8E/%E8%AE%B0%E5%BF%86%E5%AE%9E%E7%8E%B0%E4%B8%8E%E9%9D%A2%E8%AF%95%E8%AF%B4%E6%98%8E.md)) — 记忆 4 层架构、认知心理学映射、打分召回算法与 Auto-Dream
  - [评测说明.md](./架构与技术设计/评测说明.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%8A%80%E6%9C%AF%E8%AF%B4%E6%98%8E/%E8%AF%84%E6%B5%8B%E8%AF%B4%E6%98%8E.md)) — Harness 运行合同评测、FakeModelClient 确定性回放与 3 大消融实验

- **面试高频实战/**
  - [简历项目面试追问.md](./面试高频实战/简历项目面试追问.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E9%9D%A2%E8%AF%95%E9%AB%98%E9%A2%91%E5%AE%9E%E6%88%90/%E7%AE%80%E5%8E%86%E9%A1%B9%E7%9B%AE%E9%9D%A2%E8%AF%95%E8%BF%BD%E9%97%AE.md)) — 10 大硬核真实场景大厂追问与高分答题骨架
  - [面试逐字话术.md](./面试高频实战/面试逐字话术.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E9%9D%A2%E8%AF%95%E9%AB%98%E9%A2%91%E5%AE%9E%E6%88%90/%E9%9D%A2%E8%AF%95%E9%80%90%E5%AD%97%E8%AF%9D%E6%9C%AF.md)) — 1 分钟大厂自我介绍、4 大技术亮点表达与 Troubleshooting 踩坑 Debug 案例
  - [Pico源码级面试追问与技术内幕.md](./面试高频实战/Pico源码级面试追问与技术内幕.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E9%9D%A2%E8%AF%95%E9%AB%98%E9%A2%91%E5%AE%9E%E6%88%90/Pico%E6%BA%90%E7%A0%81%E7%BA%A7%E9%9D%A2%E8%AF%95%E8%BF%BD%E9%97%AE%E4%B8%8E%E6%8A%80%E6%9C%AF%E5%86%85%E5%B9%95.md)) — 底层核心代码片段 (Prompt Caching、Compact、Recall、Worker、Lock)

- **官方参考手册/**
  - [configuration.md](./官方参考手册/configuration.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/configuration.md)) — 配置合并优先级与 .pico.toml 说明
  - [memory.md](./官方参考手册/memory.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/memory.md)) — 记忆功能使用与分层结构说明
  - [sandbox.md](./官方参考手册/sandbox.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/sandbox.md)) — 命令防护沙箱与安全隔离说明
  - [skills.md](./官方参考手册/skills.md) ([绝对路径](file:///Users/chenfanghang/PycharmProjects/pico/docs/%E5%AE%98%E6%96%B9%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C/skills.md)) — Skills 技能扩展机制与结构化 SOP 说明

---

## 🗺️ 面试与复习推荐路线

1. **第一步：全局架构感知** ➡️ 阅读 [项目架构说明.md](./架构与技术设计/项目架构说明.md)，掌握两条主线（装配线与执行线）及调用链。
2. **第二步：核心模块深挖** ➡️ 深入 [记忆实现与面试说明.md](./架构与技术设计/记忆实现与面试说明.md) 与 [评测说明.md](./架构与技术设计/评测说明.md)。
3. **第三步：面试硬核实战** ➡️ 重点背诵 [简历项目面试追问.md](./面试高频实战/简历项目面试追问.md)（10 大硬核场景追问）与 [面试逐字话术.md](./面试高频实战/面试逐字话术.md)（含真实 Troubleshooting 踩坑案例）。
4. **第四步：源码细节备查** ➡️ 面试深挖底层代码时，查阅 [Pico源码级面试追问与技术内幕.md](./面试高频实战/Pico源码级面试追问与技术内幕.md)。
5. **第五步：参考手册查阅** ➡️ 参考手册请查阅 [configuration.md](./官方参考手册/configuration.md)、[memory.md](./官方参考手册/memory.md)、[sandbox.md](./官方参考手册/sandbox.md)、[skills.md](./官方参考手册/skills.md)。
