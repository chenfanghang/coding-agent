# Skills

Skill 是一段写在 markdown 文件里的可复用 prompt，可以通过 `/skill-name [args]` 调用。pico 把它展开成一次普通 session 请求，沿用同一套工具、审批和事件链路。

## 内置 skill

- `/review` — 代码审查当前改动
- `/test` — 跑测试、整理失败原因
- `/commit` — 准备 commit 消息和拆分建议
- `/simplify` — 找代码冗余并修

```bash
pico
> /review
↳ Bash(git diff) ✓
代码审查：3 处可简化，1 处有遗漏的错误处理...
```

## 加载顺序

后加载的同名 skill 覆盖前面的：

1. **内置 skill** — pico 自带
2. **用户 skill** — `~/.pico/skills/<name>/SKILL.md`
3. **项目 skill** — `<repo>/skills/<name>/SKILL.md` 或 `<repo>/.pico/skills/<name>/SKILL.md`

## 自定义一个 skill

最小例子，新建 `~/.pico/skills/deploy/SKILL.md`：

```markdown
---
name: deploy
description: 部署前清单
argument-hint: target
allowed-tools: read_file, search
---

请检查仓库是否满足部署到 $ARGUMENTS 环境的条件：

1. tests/ 通过
2. CHANGELOG 包含本次发布条目
3. 配置文件里 production endpoint 没有指向 dev
```

调用：

```text
> /deploy staging
```

`$ARGUMENTS` 会被替换为 `staging`。`${PICO_SKILL_DIR}` 会被替换为 skill 文件所在目录的绝对路径。

## Frontmatter

```yaml
---
name: skill-name              # 必填，slash 命令名
description: 一句话描述        # 必填，/skills 列表显示
when-to-use: 适用场景          # 可选，给模型的提示
argument-hint: target          # 可选，help 提示用
arguments: target=str          # 可选，命名参数
context: inline                # inline（默认）| fork
allowed-tools: read_file, search  # 可选，限制本次 skill 调用能用的工具
paths: src/*.py, tests/*.py    # 可选，把这些路径作为相关文件提示给模型
disable-model-invocation: true # 可选，只渲染 prompt 不发请求
model: claude-sonnet-4-6       # 可选，强制使用特定模型
user-invocable: true           # 可选，是否允许用户从 REPL 直接调用
---
```

## context: inline vs fork

- `inline`（默认）：skill 内容直接 append 到当前 session 的下一轮请求里。模型记得之前对话。
- `fork`：起一个隔离 session 跑 skill，主 session 不受污染。适合"和当前对话无关的一次性查询"。

## allowed-tools

如果一个 skill 只需要只读分析（如 `/review`），可以加 `allowed-tools: read_file, search, list_files`，让 skill 调用时模型看不到 write/shell。提高执行安全。

## paths

帮模型快速找到相关文件：

```yaml
paths: src/**/*.py, !src/legacy/**
```

支持 glob，前缀 `!` 表示排除。

## 调试

- `/skills` 列出所有可用 skill 和加载来源
- skill 执行时，事件流里会有 `skill_invoked` / `skill_finished`
- 用 `disable-model-invocation: true` 配合 `pico --tui` 可以在不发请求的情况下预览 skill 展开后的 prompt
