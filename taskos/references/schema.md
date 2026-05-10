# TaskOS 数据模型规范（schema）

本文件定义 TaskOS 的全部数据结构、文件格式、字段语义和操作规则。所有工作流的写操作都必须遵守本文件。

---

## 一、文件清单与格式映射

| 文件 | 格式 |
|---|---|
| `INDEX.md` | Markdown（含表格、列表、当周快照） |
| `.journal.md` / `.journal-YYYY-MM.md` | Markdown，append-only 时间戳日志 |
| `areas/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/active/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/archive/{中文名}.md` | 同上 |
| `tasks/inbox.md` | Markdown checklist |
| `tasks/active.md` | Markdown 顶部元数据 + JSONL 区块 |
| `tasks/done-YYYY-MM.md` | Markdown 顶部元数据 + JSONL 区块 |
| `reviews/YYYY-Www-review.md` | Markdown |
| `reviews/_align-log.md` | Markdown 列表，按日期分组 |
| `reviews/distill/YYYY-MM.md` | Markdown |

---

## 二、Area 文件 frontmatter

```yaml
---
type: area
name: 心理学                           # 必填，与文件名一致
created: 2026-05-10                    # 必填
status: active                         # active | dormant | retired
identity: "对应身份认同（一句话）"      # 必填
yearly_intent: "今年要交付的关键成果"   # 可空字符串
rituals:                               # 可选数组；每条必须含 desc + freq
  - desc: 精读 1 篇外刊封面文章
    freq: weekly                       # weekly | monthly
  - desc: 听完 5 期专业播客
    freq: monthly
---

（正文可选：详细描述、相关参考、子主题等）
```

---

## 三、Project 文件 frontmatter

```yaml
---
type: project
name: 精读《动力取向心理治疗》         # 必填，与文件名一致
area: 心理学                           # 必填，引用某 area.name
priority: core                         # core | normal | side
created: 2026-05-10                    # 必填
deadline: 2026-08-10                   # 必填，必须 ≤ created + 3 个月
status: active                         # active | paused | done | dropped
milestone: "完成精读 + 输出 5000 字综述"  # 必填，一句话描述里程碑
progress: 0.42                         # 0-1，用户主观给
risk: 🟢                               # 🟢 | 🟡 | 🔴，由 weekly 自动写入

# 可选：关键里程碑（防温水煮青蛙）
key_milestones:
  - 完成全书精读: pending              # pending | done
  - 输出大纲: pending
---

（正文可选：详细规划、思路、参考资料等）
```

**约束**：
- `priority: core` 全局同时存在数 ≤ 3
- `deadline - created` ≤ 90 天
- `status: done | dropped` 满 14 天自动归档到 `projects/archive/`

---

## 四、Task — JSONL 行格式

### 4.1 active.md / done-YYYY-MM.md 整体结构

```markdown
# Active Tasks                          # done-*.md 标题为 "Done Tasks YYYY-MM"
last_updated: 2026-05-10 05:18
total: 47
by_project_count:
  精读《动力取向心理治疗》: 12
  播客第 5 期录制: 8
  _orphan: 3                            # projects=[] 且非 ritual
  _ritual: 4                            # ritual 任务（含 ritual_source）

## ~~~ JSONL 区块开始 ~~~
{"id":"t-20260510-001","title":"精读第 3 章","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@阅读","est":"90min","carry":0,"due_week":"2026-W19","tier":"must","captured":"2026-05-10"}
{"id":"r-2026-W19-001","title":"精读 1 篇外刊封面文章","projects":[],"area":"心理学","context":"@阅读","est":"60min","carry":0,"due_week":"2026-W19","tier":"should","ritual_source":"心理学","ritual_desc":"精读 1 篇外刊封面文章","freq":"weekly","captured":"2026-05-10"}
## ~~~ JSONL 区块结束 ~~~
```

**by_project_count 特殊 key 判定规则**：
- `_ritual` = 任意含 `ritual_source` 字段的任务（typically `projects=[]` 且 `area != null`）
- `_orphan` = `projects=[]` 且**不含** `ritual_source` 字段的任务（例如仅挂 area 的普通 task）
- 其他 key = 任务的 `projects` 数组中每个 project 名（一个任务挂多 project 时计入每个 project 的 key）
- 不变量：`sum(by_project_count.values()) ≥ total`（多挂任务会被多次计入；如果只想要单计可以用 _orphan + _ritual + 各 project 任务数去重计算，但默认按聚合计入）

### 4.2 active.md 任务字段定义

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | ✅ | `t-YYYYMMDD-NNN` / `r-YYYY-Www-NNN` / `r-YYYY-MM-NNN` |
| `title` | string | ✅ | 任务描述（动词前置） |
| `projects` | array | ✅ | 关联项目数组（可空 []） |
| `area` | string \| null | 否 | 仅挂 area 不挂 project 时填 |
| `context` | string | 否 | 情境标签（@阅读 / @电脑 / @电话 / @外出 / @写作 …） |
| `est` | string | 否 | 估时（"90min" / "2h"） |
| `carry` | number | ✅ | 结转次数，默认 0；≥3 时 AI 主动告警 |
| `due_week` | string \| null | 否 | "2026-W19"；未排时为 null |
| `tier` | string \| null | 否 | "must" / "should" / "could"；未排时为 null |
| `ritual_source` | string | 否 | 仪式型任务的来源 area 名（如 "心理学"） |
| `ritual_desc` | string | 否 | 仪式型任务的原始 desc 文本（用于身份认证） |
| `freq` | string | 否 | 仪式型频率（"weekly" / "monthly"），仅 ritual 任务有 |
| `captured` | string | ✅ | 捕获日期（YYYY-MM-DD） |

### 4.3 done-YYYY-MM.md 额外字段

| 字段 | 必填 | 说明 |
|---|---|---|
| `completed` | ✅ | 完成或放弃的日期（YYYY-MM-DD） |
| `outcome` | ✅ | "done" 或 "dropped" |
| `week` | 否 | 完成时所属 ISO 周（用于 INDEX 当周快照"已完成本周"过滤） |

**done 文件保留原 active 时的全部字段（含 null）**，加 completed + outcome + week。

---

## 五、JSONL 区块操作规则（强制）

```
## ⚠️ JSONL 区块操作规则（不可违反）

1. JSONL 区块由两行固定字面量边界标识：
     ## ~~~ JSONL 区块开始 ~~~
     ## ~~~ JSONL 区块结束 ~~~
   不允许翻译或修改这两行。AI 只在两行之间操作。

2. 每行必须是合法 JSON 对象，且占满一行（不允许跨行）：
   - 字符串用双引号
   - 中文字符直接写不转义
   - null 值写为 null（小写）
   - 空数组写为 []

3. 添加任务（capture）：在区块结束行的前一行 append 新行。

4. 修改任务字段：
   - 用 id 定位行 → 整行重新构造 → 替换原行
   - 严禁"原地改字段"

5. 完成 / 放弃任务：
   - 从 active.md 完整删除该行
   - 直接根据 completed 日期写到对应 done-YYYY-MM.md（不存在时由 AI 自动创建）
   - 完成 outcome:"done"，放弃 outcome:"dropped"
   - done 文件保留原 active 全字段（含 null）+ completed + outcome + week

6. 解析鲁棒性：每行独立 JSON.parse，遇到坏行只跳过该行 + 在
   _align-log.md 记一笔（🔴 级别），不影响其他行。

7. 任何对 active.md / done-*.md 的写操作都强制走 .journal.md（无例外）。

8. 每次写操作后必须重算顶部元数据：
   - last_updated（当前时间）
   - total（JSONL 区块合法行数）
   - by_project_count（按 projects 字段聚合 + _orphan + _ritual）
   元数据若与 JSONL 不一致，以 JSONL 为准。
```

---

## 六、ID 编码规范

| 类型 | 格式 | 示例 |
|---|---|---|
| 普通任务 | `t-YYYYMMDD-NNN` | `t-20260510-001` |
| 周频仪式型 | `r-YYYY-Www-NNN` | `r-2026-W19-001` |
| 月频仪式型 | `r-YYYY-MM-NNN` | `r-2026-05-001` |

**ID 唯一性保证**：每次新建 ID 时，AI 必须扫描以下源以确认 NNN 序号唯一：
- `tasks/active.md`
- `tasks/inbox.md`
- 任务 captured 日期所在月的 `done-YYYY-MM.md`（即按 captured 月份匹配，通常是当月，跨月补录时则是上月）

NNN 从 001 起递增。

---

## 七、INDEX.md 结构

```markdown
# TaskOS Index
last_updated: 2026-05-10 05:18:00
last_full_rebuild: 2026-05-04 (W18 review)
last_weekly_plan: 2026-05-10
version: 287
current_week: 2026-W19

## Areas (active: 4)
- 心理学 (core: 1, normal: 2, side: 0)
- 自媒体 (core: 1, normal: 0, side: 1)

## Projects — Core (3/3) ⚠️ 已满
| name | area | risk | progress | deadline |
|---|---|---|---|---|
| 精读《...》 | 心理学 | 🟢 | 42% | 2026-08-10 |

## Projects — Normal (5)
（同上表格式）

## Projects — Side (2)
（同上表格式）

## Tasks Pool 概览
- inbox: 7 条（最久 8 天）
- active 总数: 47
- 高 carry 任务（≥3）: 2 条 ⚠️
- 滞留任务（due_week < current_week 且未完成）: 5 条 ⚠️
- by_project: 精读《...》12 / 播客 8 / _orphan 3 / _ritual 4

## 当前周 2026-W19（生成于 2026-05-10）
range: 2026-05-04 ~ 2026-05-10

### 必须做（must, ≤3）
- [ ] t-20260510-001 精读第 3 章 → 精读《...》

### 该做（should, ≤7）
- [ ] r-2026-W19-001 本周精读 1 篇外刊（仪式型 / 心理学）

### 可以做（could, ≤5）

### 已完成本周
（来源：done-YYYY-MM.md 中筛选 week == current_week 且 outcome == "done"；
若 current_week 的日期范围跨两个月份（仅 W01 跨年时发生），需同时读两个月份的 done 文件）
- [x] t-20260509-005 写文章大纲（2026-05-09）

## 上次月度蒸馏
- 2026-04（执行于 2026-05-01）
```

### INDEX 写入规则

- 每个写操作（capture / weekly plan / weekly review / rename / archive / distill / 状态变更 / 完成 / 放弃）结束时**必须**：
  - 刷新 `last_updated`
  - `version += 1`
  - 如果改变了 due_week / tier / 任务完成状态 → 重新生成"当前周"段
- weekly plan 结束时刷新 `last_weekly_plan`
- weekly review 结束时刷新 `last_full_rebuild` 并完整重建 INDEX

### "已完成本周"子段查询规则

- 来源：`done-YYYY-MM.md` 中筛选 `week == current_week` 且 `outcome == "done"`
- 若 current_week 的日期范围跨两个月份（W01 跨年场景），需同时读两个月份的 done 文件

### 启动校验

每次启动 skill 时：
1. 全量读 INDEX.md 加载到上下文
2. 三计数轻量校验：
   - active.md JSONL 行数 vs INDEX `active 总数`
   - `areas/` 目录下 .md 文件数 vs INDEX 列出的 area 数
   - `projects/active/` 目录下 .md 文件数 vs INDEX 各 priority 累加数
3. 任一不一致 → 提示用户"INDEX 可能漂移"，询问是否当场重建

### "上周" / "滞留" 计算

- "上周" = 当前日期减 7 天 → 求其 ISO 周编号（**用日期计算，不能字符串递增**）
- "滞留任务" = active.md 中 `due_week != null` 且 `due_week < current_week` 且未完成（**比较时把 ISO 周转成日期，避免字符串字典序歧义**）

### current_week 切换时机

- 周复盘时：当周快照复制到 review 文件**永久存档**，**不立即切换 current_week**
- 启动行为时：比对 INDEX.current_week 与今天 ISO 周；不一致 → 主动询问用户"要不要现在开周计划？"
  - 用户同意 → 进入 weekly plan 工作流
  - 用户拒绝 → AI 自动**只更新 INDEX.current_week 为今天 ISO 周**，并重新生成"当前周"段（基于 active.md 中 due_week == 新 current_week 的任务；可能为空），整个修正走 journal

---

## 八、`.journal.md` 操作日志格式

```markdown
## 2026-05-10 05:18:00 [in_progress] op=#287
intent: 完成 task t-20260510-003
files_planned:
  - tasks/active.md (删除该 task 的 JSONL 行)
  - tasks/done-2026-05.md (append 完整字段 + completed:2026-05-10 + outcome:done + week:2026-W19)
  - INDEX.md (重生成当前周段)

## 2026-05-10 05:18:03 [done] op=#287
```

### 启动检测
- 读 `.journal.md` 末尾
- 最后一条 `[in_progress]` 没配对 `[done]` → 提示用户"上次操作 op=#287 没完成，files_planned 是 [...]，是否：A 继续 / B 回滚 / C 标记已完成"

### 月度归档
- 每次启动时检查 `.journal.md` 中最早一条记录的月份
- 早于当月 → 把所有早于当月的内容切到 `.journal-YYYY-MM.md`，当月留在 `.journal.md`

### 触发条件
**所有写操作都走 journal，无例外**。

---

## 九、风险模型

由用户在 weekly review 时主观给 milestone 完成度（0-100%），AI 计算 risk。

```
expected_progress = (今天 - created) / (deadline - created)
actual_progress   = milestone 完成度（用户主观，0-1）
gap               = actual_progress - expected_progress

等级：
  gap ≥ -0.10                          → 🟢
  -0.30 ≤ gap < -0.10                  → 🟡
  gap < -0.30                          → 🔴
  剩余天数 < 7 且 actual_progress < 0.7  → 🔴
```

### 温水煮青蛙检测（仅当填了 key_milestones 时）
- 剩余天数 < 50% 但 key_milestones 全 pending → 强制 🔴
- 剩余天数 < 25% 但 key_milestones 完成 < 50% → 强制 🔴

### 优先级加权
- core 项目 🟡 → 提示视为 🔴
- side 项目 🔴 → 仅 weekly review 时提示

### 周复盘 side 项目 progress 跳过时的自动重算
- actual_progress 取 frontmatter 当前值（不变）
- expected_progress 用今天日期重算
- gap = actual - expected → 套等级表
- AI 写回 risk 字段，**但不更新 progress**（progress 是用户主观字段）

---

## 十、语言约定

### 必须中文
- SOP 说明、决策树、提示语、注释
- 工作流标题、章节标题
- 错误提示 / 用户对话模板

### 必须英文（机器可读性）
- YAML 键名：`type`, `name`, `area`, `priority`, `deadline`, `status`, `progress`, `risk`, `key_milestones`, `yearly_intent`, `rituals`, `identity`, `created`, `milestone`, `desc`, `freq`
- YAML 枚举值：`active / dormant / retired`、`core / normal / side`、`active / paused / done / dropped`、`pending / done`、`weekly / monthly`
- JSONL 字段名：`id / title / projects / area / context / est / carry / due_week / tier / captured / completed / outcome / week / ritual_source / ritual_desc / freq`
- JSONL 字段枚举值：`must / should / could`（tier）、`done / dropped`（outcome）、`weekly / monthly`（freq）
- ID 格式、ISO 周编号、TASKOS_ROOT 变量名、skill 自身文件名

### 用户自由
- area / project 名称：中文优先
- title / context / milestone / identity / yearly_intent / rituals.desc：自由书写
