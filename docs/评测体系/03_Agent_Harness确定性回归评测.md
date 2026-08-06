# Agent Harness 确定性回归评测

> 本评测验证的是 Pico Runtime 的工程合同：相同任务、相同初始 Workspace 和相同模型响应下，控制循环、工具治理、状态记录与 Verifier 是否稳定工作。

---

## 1. 要验证的问题

真实模型具有随机性。如果一次任务失败，仅观察最终答案无法判断是模型能力波动，还是工具解析、权限网关、状态机、Checkpoint 或完成逻辑发生回归。

Harness Regression 因此固定模型输出，重点回答：

- 工具调用能否被正确解析和执行？
- 错误调用被拒绝后，控制循环能否继续？
- 文件修改是否符合 Read-Before-Write 等治理规则？
- 上下文压缩和恢复事件是否被正确记录？
- 最终 Workspace 是否满足确定性验收条件？
- 整个任务是否在规定步数内完成？

---

## 2. 实现方式

核心入口：

- 任务定义：[`benchmarks/coding_tasks.json`](../../benchmarks/coding_tasks.json)
- 执行器：[`pico/evaluation/evaluator.py`](../../pico/evaluation/evaluator.py)
- 集成测试：[`tests/test_evaluator.py`](../../tests/test_evaluator.py)
- 运行时工件适配：[`pico/evaluation/harnessbench.py`](../../pico/evaluation/harnessbench.py)

```text
coding_tasks.json
  |  固定 Prompt、Fixture、工具范围、预算、Verifier
  v
复制全新 Fixture Workspace
  |
  v
ScriptedModelClient 提供固定响应序列
  |
  v
真实 Pico Runtime 执行控制循环与工具治理
  |
  +--------> .pico/runs/<run_id>/trace.jsonl
  +--------> .pico/runs/<run_id>/report.json
  +--------> Workspace 最终文件
  |
  v
确定性 Verifier + 预算检查
  |
  v
harness-regression-v2.json
```

这里固定的是“模型决策序列”，不是绕过 Agent Runtime。工具参数校验、执行、门禁、状态变化和工件落盘仍走真实实现。

---

## 3. 固定任务覆盖范围

当前 Benchmark 包含 12 个任务，覆盖四类 Harness 合同。

| 类别 | 代表任务 | 实验含义 |
|---|---|---|
| 文档与文本编辑 | README 和 `sample.txt` 修改 | 验证读取、Patch 与最终文件状态 |
| 工具边界恢复 | Invalid Patch、Path Escape、Repeated Read | 验证错误拒绝后仍能继续完成目标 |
| 恢复与上下文 | Context Reduction、Freshness、Workspace Mismatch | 验证 Checkpoint、重锚定与 Drift 事件 |
| Durable Memory 合同 | Promotion Accept / Reject | 验证稳定事实写入以及 Secret、临时状态拒绝 |

任务 Schema 至少包含：

| 字段 | 作用 |
|---|---|
| `id` | 稳定任务标识 |
| `prompt` | 发给 Agent 的真实请求 |
| `fixture_repo` | 冻结的初始 Workspace |
| `allowed_tools` | 本任务允许使用的工具 |
| `step_budget` | 最大执行步数 |
| `expected_artifact` | 预期外部结果说明 |
| `verifier` | 确定性成功检查 |
| `category` | 能力分类 |

---

## 4. Verifier 如何判断成功

Verifier 不相信模型的 `<final>` 文本，而是检查外部事实。例如：

- 目标文件是否包含预期修改。
- 原始错误文本是否已经消失。
- `report.json` 是否包含 Checkpoint。
- `trace.jsonl` 是否记录了 `freshness_mismatch` 或 Runtime Identity 变化。
- Durable Memory 是否只写入允许的稳定事实。
- Secret-shaped 和临时任务状态是否出现在拒绝列表中。

最终任务通过通常要求：

```text
模型完成信号
    AND
确定性 Verifier 通过
    AND
工具步数未超过 Step Budget
```

---

## 5. 当前结果

根据 2026-07-29 的完整评测报告：

| 指标 | 结果 |
|---|---:|
| 固定任务数 | 12 |
| 任务通过率 | 100% |
| Verifier 通过率 | 100% |
| 预算内完成率 | 100% |

---

## 6. 结果如何解释

这些结果可以证明：

- 当前固定合同在确定性响应下全部成立。
- 错误工具调用后的恢复路径没有破坏最终任务。
- Context、Resume 和 Durable Memory 的关键事件可被外部工件验证。
- 回归任务均能在预设步数预算内结束。

这些结果不能证明：

- 任意真实模型都能完成这 12 个任务。
- Pico 已覆盖所有真实代码仓库复杂度。
- 100% 的确定性回归等于真实用户任务 100% 成功。

真实模型效果需要结合 [真实模型与真人场景验收](./08_真实模型与真人场景验收.md) 解读。

---

## 7. 面试表达

> “我把模型随机性和 Harness 工程逻辑分开评估。固定 Benchmark 中，每个任务都有冻结的 Workspace、允许工具、Step Budget 和确定性 Verifier；模型侧使用固定响应序列，但工具解析、权限检查、状态机、Checkpoint 和工件落盘仍走真实 Runtime。这样一旦代码改动造成回归，可以直接定位到 Harness 合同，而不是被 Provider 波动干扰。目前 12 个任务的任务通过率、Verifier 通过率和预算内完成率均为 100%。这个结果证明运行时合同稳定，但我不会把它解释成真实模型能力上限，所以另外还跑了 DeepSeek Live 和真人场景验收。”

---

> **上一篇：** [02_统一实验协议与指标口径.md](./02_统一实验协议与指标口径.md)  
> **下一篇：** [04_上下文编排与成本评测.md](./04_上下文编排与成本评测.md)
