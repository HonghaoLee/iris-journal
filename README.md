# iris-journal

AI 会话结束即失忆，git commit 只记录"改了什么"，不记录"为什么改"和"踩了哪些坑"。iris-journal 一句话触发，将每次会话的决策与产出结构化归档，让上下文跨会话留存。

## 安装

```bash
npx skills add https://github.com/HonghaoLee/iris-journal
```

## 触发方式

在 Claude Code 会话结束时，直接说：

- 「帮我归档今天的工作」
- 「写个今天的 recap」
- 「记录一下刚才做的事」
- 「今天工作结束，存档一下」

首次使用会自动初始化，确认用户名和日志目录（默认为项目根目录下的 `journals/`）。

## 日志结构

```
journals/
├── journal-config.json
├── index/
│   └── {USERNAME}-YYYY-MM.json
└── YYYY-MM/
    └── {USERNAME}-YYYY-MM-DD.md
```

每条日志按时间窗口组织，同一天多次触发自动追加，不覆盖：

```markdown
## 时间窗口 #1 · 14:30:00

### 完成的任务
### 关键决策
### 遇到的问题与解决方案
### 未完成事项 / 下一步计划
```

## 不触发的场景

涉及 git commit、分支、diff、PR、提交信息、changelog 的上下文均不触发，即使出现「总结」「记录」等词。

## 配置

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `USERNAME` | 日志文件所属用户名 | 初始化时确认 |
| `JOURNAL_DIR` | 日志根目录绝对路径 | `{项目根目录}/journals/` |
