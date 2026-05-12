# Workflow: Capture（捕获分层 + inbox 整理）

负责把用户的自然语言任务输入归类到正确位置。涵盖两类操作：
1. **新建任务**（用户说"记一下…"、"加任务"等）
2. **inbox 整理**（用户说"把 inbox 那条 X 归到 Y"等）

---

## 一、触发词

- 记一下 / 加一个任务 / 加任务 / 帮我归类 / 给我加进任务列表
- 把 inbox 那条 X 归到 Y / 把 X 改挂到 Y 项目 / X 归到 Y

---

## 二、新建任务的决策树

> 用户输入一段自然语言任务描述时执行此流程。

### 步骤 1：动词前置改写

把用户的原始描述改写为**动词开头的祈使句**，消除执行阻力。

| 原始描述 | 改写后 |
|---|---|
| "下周联系张老师" | "联系张老师确认督导时间" |
| "想读那本书第三章" | "精读《动力取向心理治疗》第 3 章" |
| "外刊那篇文章要看" | "精读 The Economist 本周封面文章" |

如果用户描述本身已是动词前置 → 不改。

### 步骤 2：归类决策树

按以下顺序判断：

1. **命中某个 active project**（用户描述中出现 project 名 / 关键词匹配某 active project 的 milestone）
   → 写到 `tasks/active.md`，`projects: [命中的 project 名]`，`area: null`

2. **只命中某个 area**（关键词匹配某 area 的 identity / yearly_intent / rituals，但找不到具体 project）
   → 写到 `tasks/active.md`，`projects: []`，`area: 命中的 area 名`

3. **都不命中**（描述模糊，AI 也判断不出）
   → 写到 `tasks/inbox.md`，等周计划时由用户手动整理

### 步骤 3：补全字段

无论写到 active.md 还是 inbox.md，必须填齐以下字段：
- `id`：按规则生成（见 schema.md 第六节）
- `title`：动词前置改写后的描述
- `captured`：今天日期（YYYY-MM-DD）
- `context`（可选）：从用户描述推断（如"打电话"→ @电话；"读书"→ @阅读）
- `est`（可选）：从用户描述推断或追问

active.md 还需补：
- `projects`、`area`（按决策树）
- `carry: 0`、`due_week: null`、`tier: null`（capture 时不排期）

### 步骤 4：journal + Mini-Check + INDEX

- 写 `.journal.md`（单行格式：`[时间] #NNN done | capture t-XXXXXXXX-NNN → active`）
- 跑 Mini-Check（详见 SKILL.md Mini-Check 表）
- 刷新 INDEX：last_updated / version / 相应 tasks pool 概览

---

## 三、inbox 整理（重新分类）

### 触发场景
- 用户主动说"把 inbox 那条 X 归到 Y"
- 用户在 weekly plan 的"强制清 inbox"步骤中逐条处理
- AI 在月度蒸馏 / 完整性扫描时建议整理

### 操作步骤

1. **定位**：在 `tasks/inbox.md` 中找到该 task（按 ID 或 title 关键词）
2. **保留原 ID**（不重新生成，便于追溯）
3. **从 inbox.md 删除**该条
4. **append 到 active.md**，补全 active 必需字段：
   - `projects`：用户/AI 现场判定
   - `area`：如有
   - `carry: 0`、`due_week: null`、`tier: null`
   - 保留 inbox 已有字段：title / context / captured（注意：inbox 中 captured 字段在原 markdown 列表里）
5. **journal + Mini-Check + INDEX**

### 三种归宿
| 用户决策 | 操作 |
|---|---|
| 挂某 project | 写 active.md，projects: [X] |
| 仅挂 area（暂无 project） | 写 active.md，projects: []，area: X |
| 删除 | 从 inbox.md 删除，不写 done.md（inbox 阶段未"做过"，没有归档价值） |

---

## 四、特殊情况

### 4.1 一个动作同时推进多个 project
用户说："我想写一篇关于客体关系的文章" — 这同时是「精读《动力取向心理治疗》」的输出和「自媒体第 5 期」的素材。

→ `projects: ["精读《动力取向心理治疗》", "自媒体第 5 期录制"]`

完成时只勾选一次（删除一行），但 INDEX 中两个 project 的"关联任务数"都会减 1。

### 4.2 用户描述中含日期 / 时间提示
用户说："明天上午精读第 3 章" — capture 时**不要自动设 due_week**（capture 阶段不排期）。如果用户明确说"放进本周"，可以同时设 `due_week: current_week, tier: should`。

### 4.3 ritual 任务不通过 capture 创建
ritual 任务由 weekly plan 自动生成（详见 workflow-weekly.md "Ritual 任务的特殊处理"），不通过 capture。如果用户说"加一条本周精读外刊"——AI 应判断 area 的 rituals 字段中是否已有此项，若有则提示"这是 area 的 ritual，本周开周计划时会自动生成，不需要手动 capture"。

### 4.4 输入是多条任务一次性批量
用户说："记一下：A、B、C 三件事" — AI 应**逐条**走决策树，分别写入对应位置。每条独立 journal 事务（或合并为一次 journal 事务列出全部 files_planned，再统一执行）。

---

## 五、Mini-Check（capture 后立即跑）

| 操作 | 检查 |
|---|---|
| Capture 到 active.md | id 全局唯一；引用的 project 是否存在；动词前置改写完成；INDEX 已刷新 |
| Capture 到 inbox.md | id 全局唯一；动词前置改写完成；captured 已填；INDEX 已刷新 |
| Inbox 整理（移到 active） | inbox.md 该行已删；active.md 已 append 且 ID 保留；缺失字段已补全；INDEX 已刷新 |

不一致 → 立即提示用户 + 给修复建议（不擅自修复）。

---

## 六、AI 与用户对话模板（参考）

### 命中 project 时
> 已记录："精读第 3 章"。挂在「精读《动力取向心理治疗》」项目下，归到 active.md（未排期）。

### 只命中 area 时
> 已记录："联系张老师确认督导时间"。这条暂时只挂在「心理学」area，未关联具体项目。要不要给它挂个项目？还是暂时这样？

### 都不命中时
> "X" 我没法判断挂哪里，先放进 inbox 了，下周开周计划时一起整理。

### inbox 整理时
> 把 inbox 里 "X" 挂到「Y」项目下了。原 ID t-XXX 保留，captured 日期不变。
