# 分层记忆 + Auto-dream

pico 的记忆系统让 agent **跨 session 保持对项目的认知**。不是把整个对话历史塞回 prompt，而是分四层落地，每层有自己的生命周期。

## 为什么需要分层

把一次 session 的所有事都喂给下次对话——上下文会爆。完全不记——agent 永远是第一次见你。分层的想法是：

- **当前任务相关**：保留高保真，但只在本 session 内有效。
- **长期可复用**：经过提炼，跨 session 持久化。
- **零散观察**：先 append-only 写日志，定期再整理。

## 四层结构

```
.pico/memory/
├── MEMORY.md                       # 索引：列出哪些 topic 文件值得看
├── topics/                         # durable memory（4 类）
│   ├── user-preferences.md
│   ├── project-conventions.md
│   ├── key-decisions.md
│   └── dependency-facts.md
├── logs/                           # daily logs
│   └── YYYY/MM/YYYY-MM-DD.md       # append-only
└── .consolidate-lock               # auto-dream 锁文件 + 上次整合时间戳
```

加上 **working memory**（保存在 session JSON 的 `memory` 字段里），一共四层：

| 层 | 生命周期 | 内容 | 注入 prompt？ |
|----|---------|------|---------------|
| **working memory** | session 内 | 当前任务摘要 + 最近接触的文件 + 文件短摘要 | 是 |
| **daily logs** | 永久 append-only | 当天的零散观察、`/remember` 写入 | 否（除非整合后） |
| **durable topics** | 长期，可更新 | 经过 dream 整合的稳定事实 | 是（通过 MEMORY.md 索引） |
| **MEMORY.md** | 长期 | topic 文件的索引（不超过 200 行） | 是 |

## 4 类 durable topic

dream 整合时只往这四个文件里写：

- `user-preferences` — 用户的角色、知识水平、协作偏好
- `project-conventions` — 仓库约定、构建工具、命名风格
- `key-decisions` — 长期生效的设计决策和理由
- `dependency-facts` — 关键依赖的版本、行为、踩过的坑

## 写入路径

### `/remember <text>` — 一行写入 daily log

```text
> /remember 这个项目用 pytest 不用 unittest，并发测试用 pytest-xdist
Saved to daily log.
```

### `<memory>...</memory>` — agent 在 final answer 里自动追加

模型在回答里裹一对 `<memory>` 标签，pico 自动 append 到当天的 daily log。

### 后台 auto-dream — 自动整合

满足以下条件后台触发：

- 距上次整合 >= 24 小时（`--dream-interval`）
- 至少有 5 个新 session（`--dream-min-sessions`）
- 当前没有正在跑的 dream

后台启一个隔离的 pico 实例（write_scope 限制在 `.pico/memory/`），把 daily log + 最近 session ID 一起喂给模型，让它写 / 更新 topic 文件和 MEMORY.md。

### `/dream` — 手动触发

不想等后台：

```text
> /dream
Consolidation complete. Wrote 2 topic updates, refreshed index.
```

## 读取路径

每轮 prompt 自动注入两段：

1. **memory section** — working memory + MEMORY.md 索引（让模型知道有哪些长期记忆可查）
2. **relevant_memory section** — 根据当前用户请求做关键词检索，从 daily log 和 topic 里挑最相关的 3 条

模型也可以手动 `read_file .pico/memory/topics/<name>.md` 读完整 topic。

## 用户可见命令

| 命令 | 说明 |
|------|------|
| `/memory` | 显示 MEMORY.md 索引 |
| `/working-memory` | 显示当前 session 的工作记忆 |
| `/remember <text>` | 追加一条到 daily log |
| `/dream` | 立即整合 daily log → topic |

## 关闭

不需要 memory 的场景：

```bash
pico --no-auto-dream     # 只关 auto-dream，保留 /remember /dream
```

或者在 toml / 启动时设 `feature_flags.memory = false`，但**不推荐**——这是 pico 区别于其他 coding agent 的核心能力。

## 文件级 freshness 保护

在 patch_file / write_file 之前，pico 会检查"是否最近 read 过这个文件"（通过 sha256 freshness）。如果没读过就改，会被 `prior_read_required` 拒绝。这层保护和 memory feature flag **解耦**——即便 memory 关闭，read freshness 也仍然追踪，避免 agent 改盲文件。

## 故障排查

| 现象 | 原因 / 解决 |
|------|-------------|
| `/dream` 输出 `nothing to consolidate` | daily log 是空的，先 `/remember` 几条 |
| auto-dream 不触发 | 检查 `.pico/memory/.consolidate-lock` 的 mtime，距上次 24h 没到 |
| topic 文件没更新但 dream 说成功 | 早期版本的已知 bug，已在 2026-05 修复（freshness 追踪从 memory feature flag 解耦） |
| MEMORY.md 太长 | dream 会自动裁剪到 200 行；手动 `/compact` 也可以 |

## 推荐的工作流

1. 第一次进项目：让 pico 跑 `/skills`、看 README、用 `/remember` 写下 1-2 条该仓库的关键约定。
2. 每天工作结束：`/dream` 一次，把当天观察沉淀。
3. 切换分支或一段时间没用：直接 `pico --resume latest`，让它从工作记忆 + topic 里恢复上下文。

记忆只在本地，**不会上传**。删除 `.pico/memory/` 就回到第一次见你的状态。
