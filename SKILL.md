---
name: iris-journal
description: "用于归档今天的工作成果、记录本次会话产出、写 recap 或总结完成的任务与决策。用户说「记录一下」「归档今天的」「总结今天的工作/产出」「存档一下」时触发，即使未提到「日志」。支持同日多次追加，日志写入项目目录的 journals/ 下。排除 git 命令上下文：若对话涉及 git commit、分支、diff、PR、提交信息、changelog，无论是否出现「总结」「变更」「记录」等词，均不触发。同样不适用于：读取已有日志、个人生活日记、跨天汇总报告。"
---

# iris-journal

AI 会话天然无记忆，每次对话结束后工作上下文即刻消失。git commit 只记录代码变更，不保留「为什么这么改」「踩了哪些坑」「下一步打算做什么」。iris-journal 在会话结束时一句话触发，将决策、产出、问题自动结构化归档，让工作断点跨会话可追溯。

每日工作日志生成与归档技能。以本地文件为唯一真相源，用户可自行配置远程同步。

---

## 内嵌模板

### journal-template.md

新建日志文件和追加时间窗口均基于此模板的时间窗口结构，通过占位符替换生成：

```markdown
# 工作日志 · {YYYY-MM-DD}

**作者**：{USERNAME}
**日期**：{YYYY-MM-DD}
**创建时间**：{ISO-DATETIME}

---

## 时间窗口 #{N} · {HH:MM:SS}

### 完成的任务

- 

### 关键决策

- 

### 遇到的问题与解决方案

- 

### 未完成事项 / 下一步计划

- 

---
```

### journal-config-template.json

```json
{
  "USERNAME": "",
  "JOURNAL_DIR": "",
  "initialized": true
}
```

### journal-index-template.json

```json
{
  "last_updated": "{ISO-DATETIME}",
  "journals": {
    "{USERNAME}": {}
  }
}
```

---

## 模板占位符

所有模板文件中的占位符在生成时替换为实际值：

| 占位符 | 替换值 | 示例 |
|--------|--------|------|
| `{YYYY-MM-DD}` | 当前日期 | `2026-07-06` |
| `{USERNAME}` | config 中的 USERNAME | `lihonghao` |
| `{ISO-DATETIME}` | ISO 8601 格式当前时间 | `2026-07-06T14:30:00+08:00` |
| `{N}` | 时间窗口序号，从 1 开始 | `1` |
| `{HH:MM:SS}` | 当前时间 | `14:30:00` |
| `{YYYY-MM}` | 当前年月 | `2026-07` |

---

## 配置参数

| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `USERNAME` | string | 日记文件所属用户名，用于目录和索引分组；确认后不建议修改 | 无（必填）|
| `JOURNAL_DIR` | string | 日记根目录绝对路径，例如 `/Users/alice/journals` | `~/journals/` |
| `initialized` | boolean | 初始化完成标记，由初始化流程自动写入；请勿手动改为 `false` | `true`（写入后）|

配置文件位于 `JOURNAL_DIR/journal-config.json`，可参考 `references/journal-config-template.json` 手动修改。

---

## 使用示例

### 触发对话示例

| 用户说的话 | 技能动作 |
|-----------|---------|
| 「帮我归档今天的工作」 | 进入日记生成流程，收集上下文后写入当日日志文件 |
| 「写个今天的 recap」 | 同上，生成时间窗口 #1（或 append 新窗口）|
| 「总结今天的变更」 | 从会话历史提取代码变更，填入完成任务模块 |
| 「记录一下刚才解决的那个 bug」 | 追加新时间窗口，重点填充「遇到的问题与解决方案」|
| 「今天工作结束，存档一下」 | 生成完整日志并展示文件路径与条目摘要 |

**不会触发的情况**（不应误触发）：

| 用户说的话 | 原因 | 拒识机制 |
|-----------|------|---------|
| `git commit -m "fix bug"` | git 版本控制操作 | 上下文含 git 命令/分支/diff 关键词时整体判定为代码提交意图，即使出现「总结」「变更」也不触发 |
| 「帮我写今天的提交信息」 | 生成 commit message，非工作日志 | 「提交信息」「commit message」上下文 → 版本控制意图 |
| 「总结这次 PR 的变更」 | 代码 review/PR 上下文 | PR/分支/diff 上下文 → 版本控制意图 |
| 「帮我写一篇技术博客」 | 创作意图，非工作日志 | 无归档/记录意图信号 |
| 「打开昨天的日志看看」 | 读取/查看意图，非生成/归档 | 「打开」「查看」「读取」+ 已有日志 → 读操作，不触发写入 |
| 「导出日志为 PDF」 | 格式转换需求 | 超出本技能范围，无文件写入意图 |

### 日志文件样例（create 模式）

```markdown
# 工作日志 · 2026-07-06

**作者**：lihonghao
**日期**：2026-07-06
**创建时间**：2026-07-06T14:30:00+08:00

---

## 时间窗口 #1 · 14:30:00

### 完成的任务

- 修复了 iris-journal 技能中 append 模式索引更新 bug
- 补充了 SKILL.md 的使用示例和可靠性说明章节

### 关键决策

- 将配置参数说明从 JSON _comment 迁移到 SKILL.md 正文，提升可发现性

### 遇到的问题与解决方案

- 问题：JOURNAL_DIR 路径含空格时 JSON 写入失败；解决：使用绝对路径并加引号

### 未完成事项 / 下一步计划

- 推送 GitHub 并同步平台版本

---
```

### 日志文件样例（append 模式 — 同日第二次触发后）

```markdown
# 工作日志 · 2026-07-06

**作者**：lihonghao
**日期**：2026-07-06
**创建时间**：2026-07-06T14:30:00+08:00

---

## 时间窗口 #1 · 14:30:00

（第一次内容，保持不变）

---

## 时间窗口 #2 · 18:15:00

### 完成的任务

- 补充了 append 模式和索引文件的输出样例
- 修复了 SKILL.md 中错误的代码块标注

### 关键决策

- 选择在原文件末尾追加而非新建文件，保持单日日志聚合

### 遇到的问题与解决方案

- 无

### 未完成事项 / 下一步计划

- 调用平台 API 更新版本

---
```

### 索引文件样例（填充后）

`index/lihonghao-2026-07.json` 两次写入后的实际内容：

```json
{
  "last_updated": "2026-07-06T18:15:00+08:00",
  "journals": {
    "lihonghao": {
      "2026-07-06": {
        "file": "2026-07/lihonghao-2026-07-06.md",
        "created_at": "2026-07-06T14:30:00+08:00",
        "last_appended_at": "2026-07-06T18:15:00+08:00",
        "windows": [
          { "index": 1, "time": "14:30:00", "summary": "修复 iris-journal append 模式索引更新 bug，补充 SKILL.md 使用示例..." },
          { "index": 2, "time": "18:15:00", "summary": "补充 append 模式和索引文件输出样例，修复代码块标注..." }
        ]
      }
    }
  }
}
```

---

## 路径约定

```text
JOURNAL_DIR/
├── journal-config.json
├── index/
|   └── {USERNAME}-YYYY-MM.json
└── YYYY-MM/
    └── {USERNAME}-YYYY-MM-DD.md
```

默认 `JOURNAL_DIR`：当前工作目录下的 `journals/` 子目录（即 `{CWD}/journals/`）。

> ⚠️ **不要使用 `~/journals/` 等用户主目录路径。** 日志应归档在 Agent 的工作目录（项目根目录）下，与 `CLAUDE.md` 同级，便于随项目管理和同步。

---

## 初始化流程

### S0 入口判断

**每次调用时，按以下顺序定位已有配置：**

1. 读取全局指针文件 `~/.iris-journal`，取其中记录的 config 绝对路径
2. 若指针不存在，探测以下路径：
   - `{CWD}/journals/journal-config.json`
   - `{CWD}/journal-config.json`

读取找到的 config，验证 `initialized` 为 `true` 且 `USERNAME` / `JOURNAL_DIR` 非空 → 跳过初始化，直接进入日记生成。

> ⚠️ **不得跳过探测直接走初始化**。新会话无记忆，但 config 文件持久存在于磁盘，必须主动查找。全局指针和探测路径均未命中时，才触发完整初始化流程。

### S1 识别用户名

> ⚠️ USERNAME 是日记文件名核心字段，**不做猜测，必须明确确认**。

按优先级依次尝试：
1. 读取项目 `USER.md`，提取用户名字段
2. 直接询问用户输入

展示候选值给用户确认，确认后写入配置：`USERNAME`

### S2 确认存储目录

推荐默认目录：当前工作目录下的 `journals/`（即 `{CWD}/journals/`），展示绝对路径给用户确认，允许手动更改。

> ⚠️ 默认值应为 Agent 工作目录下的子目录，**不得默认为 `~/journals/` 等用户主目录路径**。

- **目录不存在** → 提示将自动创建（正常路径）
- **目录已存在且有文件** → 标注为「重新初始化或目录冲突」场景，列出现有文件，询问：合并使用 / 指定其他路径

确认后写入配置：`JOURNAL_DIR`

### S3 验证

依次验证以下各项：
1. `USERNAME` / `JOURNAL_DIR` / `initialized` 字段均非空
2. `JOURNAL_DIR` 可写（必要时创建目录）
3. 写入 `JOURNAL_DIR/journal-config.json`（含 `"initialized": true`）
4. **写入全局指针**：将 `JOURNAL_DIR/journal-config.json` 的绝对路径写入 `~/.iris-journal`（单行纯文本）
5. 若 `JOURNAL_DIR/index/` 目录不存在则创建。读取 `references/journal-index-template.json` 内容，替换 `{USERNAME}` 和 `{ISO-DATETIME}` 占位符后写入 `JOURNAL_DIR/index/{USERNAME}-YYYY-MM.json`（其中 `YYYY-MM` 为当前年月）
6. 展示配置摘要

---

## 日记生成流程

### 第 1 步：读取配置

按顺序定位 config 文件：
1. 读取全局指针 `~/.iris-journal`，取其中记录的 config 绝对路径
2. 若指针不存在，探测 `{CWD}/journals/journal-config.json`、`{CWD}/journal-config.json`

取第一个存在且有效的文件，校验 `USERNAME` 和 `JOURNAL_DIR` 非空。若均未找到，终止并提示用户先触发初始化。

### 第 2 步：收集上下文

从当前会话历史中提取，按以下规则扫描：

1. **完成的任务**：扫描已执行的工具调用（Edit / Write / Bash），归纳实际完成的操作
2. **关键决策**：识别「因为…所以…」「决定用…而不是…」「选择…而非…」等句式，提取决策及原因
3. **遇到的问题与解决方案**：识别报错信息、「不行」「失败」「换个方式」等信号词，提取问题和解法
4. **未完成事项**：识别「还差…」「下次…」「TODO」「待处理」等，提取下一步计划

> 以上各类内容不足时，**主动逐项询问用户补充**，禁止用「暂无」「-」占位。

### 第 3 步：判断 append 或 create

读取 `JOURNAL_DIR/index/{USERNAME}-YYYY-MM.json`（`YYYY-MM` 为当前年月）：

- **文件不存在** → create 模式（并将在第 4 步中创建该索引文件）
- **文件存在** → 检查 `journals[{USERNAME}][{YYYY-MM-DD}]` 条目：
  - 无条目 → create 模式
  - 有条目 → append 模式

### 第 4 步：生成日记

#### create 模式

1. **确定路径**：计算目标文件路径 `JOURNAL_DIR/YYYY-MM/{USERNAME}-YYYY-MM-DD.md`
2. **读取模板**：读取 `references/journal-template.md` 内容
3. **替换占位符**：将模板中的占位符替换为实际值：
   - `{YYYY-MM-DD}` → 当前日期
   - `{USERNAME}` → 配置中的用户名
   - `{ISO-DATETIME}` → 当前 ISO 格式时间
   - `{N}` → `1`（首个时间窗口）
   - `{HH:MM:SS}` → 当前时间
4. **填充内容**：在各模块下填入第 2 步收集的上下文信息，每条以 `- ` 开头
5. **创建目录**：若 `JOURNAL_DIR/YYYY-MM/` 不存在则创建
6. **写入文件**：将完整内容写入目标路径
7. **更新索引**：在 `JOURNAL_DIR/index/{USERNAME}-YYYY-MM.json` 中：
   - 若索引文件不存在，先读取 `references/journal-index-template.json`，替换占位符并创建文件
   - 在 `journals[{USERNAME}]` 下新增以 `{YYYY-MM-DD}` 为键的条目：
     ```json
     {
       "file": "YYYY-MM/{USERNAME}-YYYY-MM-DD.md",
       "created_at": "{ISO-DATETIME}",
       "last_appended_at": "{ISO-DATETIME}",
       "windows": [
         {
           "index": 1,
           "time": "HH:MM:SS",
           "summary": "优先取「完成的任务」第一条，不足时补「关键决策」第一条，确保是完整句子，截断在句尾标点处，最长 200 字符"
         }
       ]
     }
     ```

#### append 模式

1. **确定路径**：从索引文件 `JOURNAL_DIR/index/{USERNAME}-YYYY-MM.json` 中读取 `journals[{USERNAME}][{YYYY-MM-DD}].file`，拼接为完整路径
2. **读取现有文件**：加载目标 `.md` 文件完整内容
3. **计算窗口序号**：当前 `windows` 数组长度 + 1，即为本次时间窗口编号 `{N}`
4. **追加内容**：取 `journal-template.md` 中的时间窗口结构（`## 时间窗口 #{N}` 起至末尾 `---`），替换占位符后追加到文件末尾，**不重复文件头部（标题、作者、日期）**：

   ```markdown
   ## 时间窗口 #{N} · {HH:MM:SS}

   ### 完成的任务

   - （第 2 步收集的内容）

   ### 关键决策

   - （第 2 步收集的内容）

   ### 遇到的问题与解决方案

   - （第 2 步收集的内容）

   ### 未完成事项 / 下一步计划

   - （第 2 步收集的内容）

   ---
   ```
5. **更新索引**：在 `JOURNAL_DIR/index/{USERNAME}-YYYY-MM.json` 对应日期的条目中：
   - 将 `last_appended_at` 更新为当前 ISO 时间
   - 在 `windows` 数组末尾追加新对象：
     ```json
     {
       "index": {N},
       "time": "{HH:MM:SS}",
       "summary": "优先取「完成的任务」第一条，不足时补「关键决策」第一条，截断在句尾标点处，最长 200 字符"
     }
     ```
   - 确保 `windows` 数组按 `index` 升序排列，`file`、`created_at` 字段保持不变

### 第 5 步：输出结果摘要

向用户展示：
- 日记文件路径
- 本次写入的时间窗口
- 今日已有条目数（第 N 次记录）

---

> 本地 `JOURNAL_DIR` 为唯一真相源，Agent可以引导用户可自行搭配 Git、feishu-cli 等工具实现远程同步。

---

## 已知限制与错误处理

### 失败场景一览

| 场景 | 行为 | 恢复方式 |
|------|------|---------|
| `journal-config.json` 不存在 | 先读全局指针 `~/.iris-journal`，若指针也不存在则触发初始化 | 按初始化提示操作即可 |
| `JOURNAL_DIR` 不可写 | 报错并说明路径；终止写入 | 检查目录权限：`ls -la ~/journals/` |
| `USERNAME` 字段为空 | 拒绝继续，提示「未完成初始化，请先运行初始化流程」 | 重新触发技能，完成 S1-S3 |
| `initialized` 为 `false` | 与「config 不存在」同等处理，触发初始化 | 同上 |
| 同一天多次调用 | 自动进入 Append 模式，追加新时间窗口（#2、#3…） | 无需操作，正常行为 |
| `JOURNAL_DIR` 不存在（首次） | 自动创建目录及 `index/` 子目录 | 无需操作，正常行为 |
| 索引文件不存在（首次） | 按 `journal-index-template.json` 自动创建 | 无需操作，正常行为 |
| 会话上下文内容不足 | **主动询问用户**，不猜测或用占位符填充 | 用户补充描述后继续 |

### 适用范围

- **适用**：记录当次 AI 会话中的工作产出、决策、问题与计划
- **不适用**：查看/检索历史日志（本技能只写不读）、格式转换（PDF/Excel）、跨会话内容汇总、自动定时归档（需用户主动触发）

### 输入要求

- `USERNAME`：任意非空字符串，建议与系统用户名一致；确认后不可更改（已有文件按旧名索引）
- `JOURNAL_DIR`：有效的绝对路径，确保当前用户对该目录有读写权限