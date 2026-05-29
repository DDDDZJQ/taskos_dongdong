# TaskOS 数据模型规范（schema）

本文件定义 TaskOS 的全部数据结构、文件格式、字段语义和操作规则。所有工作流的写操作都必须遵守本文件。

---

## 一、文件清单与格式映射

| 文件 | 格式 |
|---|---|
| `INDEX.md` | Markdown（含表格、列表、当周快照） |
| `.journal.md` / `.journal-YYYY-MM.md` | 单行日志格式（append-only） |
| `areas/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/active/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/archive/{中文名}.md` | 同上 |
| `tasks/inbox.md` | Markdown checklist |
| **`tasks/active.md`** | **标题 + JSONL 区块** |
| **`tasks/done-YYYY-MM.md`** | **标题 + JSONL 区块** |
| `reviews/YYYY-Www.md` | Markdown + YAML frontmatter（结构化周快照） |

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
justification: "简述为什么这个项目值得被 TaskOS 管理"  # v1.2.2 新增，通过 Gatekeeper 后写入

# 可选：关键里程碑（防温水煮青蛙）
key_milestones:
  - 完成全书精读: pending              # pending | done
  - 输出大纲: pending
---

（正文可选：详细规划、思路、参考资料等）

正文推荐结构：
- ## 项目目标
- ## 与 area 的关系
- ## 拆解思路
- ## 关键决策点
- ## 偏好（v1.2.1 新增：用户对该项目的个人偏好，AI 做资源推荐和任务拆解时参考）
- ## 备注
```

**约束**：
- `priority: core` 全局同时存在数 ≤ 3（strategy project 不计入）
- `deadline - created` ≤ 90 天
- `status: done | dropped` 满 14 天自动归档到 `projects/archive/`

---

### Strategy Project frontmatter（v1.2 新增）

Strategy 类型的 project 使用不同的 frontmatter 字段：

```yaml
---
type: strategy
name: "[strategy] 两年达成雅思 7.5"
goal: 雅思总分 7.5（各单项 ≥ 6.5）
target_date: 2028-05
created: 2026-05-11
last_reviewed: 2026-05-11
status: active
area: 英语学习
priority: core
progress: 0.0
risk: 🟢
---
```

**与普通 project 的差异**：
- `type: strategy`（普通 project 为 `project` 或无 type 字段）
- 使用 `goal` 替代 `milestone`
- 使用 `target_date` 替代 `deadline`，**不受 90 天约束**
- 新增 `last_reviewed` 字段（上次检视日期）
- **不参与风险模型计算**（gap 公式对长期目标无意义）
- **不占用 core ≤ 3 槽位**（strategy 是长期容器，不是 90 天聚焦项目）
- 进度公式：已完成阶段数 / 总阶段数

**子 project 新增字段**：

从 strategy 拆出的普通 project 需在 frontmatter 添加：

```yaml
parent_strategy: "[strategy] 两年达成雅思 7.5"
strategy_phase: 1
```

- `parent_strategy`：指向父 strategy project 的 name 字段
- `strategy_phase`：属于路线图第几阶段（从 1 开始）
- 子 project 本身是普通 project，正常参与风险模型和 deadline 约束

---

## 四、Task — JSONL 行格式

### 4.1 active.md / done-YYYY-MM.md 整体结构

```markdown
# Active Tasks
## ~~~ JSONL 区块开始 ~~~
{"id":"t-20260510-001","title":"精读第 3 章","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@阅读","est":"90min","carry":0,"due_week":"2026-W19","tier":"must","captured":"2026-05-10"}
{"id":"r-2026-W19-001","title":"精读 1 篇外刊封面文章","projects":[],"area":"心理学","context":"@阅读","est":"60min","carry":0,"due_week":"2026-W19","tier":"should","ritual_source":"心理学","ritual_desc":"精读 1 篇外刊封面文章","freq":"weekly","captured":"2026-05-10"}
## ~~~ JSONL 区块结束 ~~~
```

> done-YYYY-MM.md 标题为 `# Done Tasks YYYY-MM`，其余结构相同。
> v1.1 变更：去掉 `last_updated` / `total` / `by_project_count` 顶部元数据。AI 需要计数时直接数 JSONL 行。

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
| `status` | string | 否 | `"not_started"` / `"in_progress"` / `"blocked"`；默认 not_started；旧行无此字段视为 not_started |
| `note` | string \| null | 否 | 自由文本进度细节（如 "110/200"）；每次更新覆盖旧值；旧行无此字段视为 null |

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
   .journal.md 记一笔 [align]（格式：[时间] #NNN align | 坏行 active.md line X），不影响其他行。

7. 任何对 active.md / done-*.md 的写操作都强制走 .journal.md（无例外）。

8. JSONL 区块边界标识符是固定字面量，不允许翻译或修改。
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
energy_this_week: normal               # 本周精力状态（high / normal / low）
work_hours_this_week: 42               # v1.2.5 新增：本周全职工时（小时），每次开周计划时更新
weekly_est_limit: 12h                  # 每周工作量软上限（用户可设）
weekly_est_limit_source: auto          # v1.2.5 新增：auto = AI 双锚点推荐 | manual = 用户手动设定
data_schema: 1.3.0                     # 当前数据 schema 版本
proactive:                             # v1.2 新增：主动规划开关
  nudge: on                            # on | off
  strategy: on                         # on | off

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

## Strategy Projects（v1.2 新增）
| name | area | progress | target_date | last_reviewed |
|---|---|---|---|---|
| [strategy] 两年达成雅思 7.5 | 英语学习 | 25% | 2028-05 | 2026-08-15 |

## Tasks Pool 概览
- inbox: 7 条（最久 8 天）
- active 总数: 47
- 本周排期: 10 条（must: 3, should: 5, could: 2）
- 高 carry 任务（≥3）: 2 条 ⚠️
- 滞留任务（due_week < current_week 且未完成）: 5 条 ⚠️
- by_project: 精读《...》12 / 播客 8 / _orphan 3 / _ritual 4

## Nudge 冷却（v1.2 新增）
- core_stall_「项目名」: 2026-W21
- carry_high: 2026-W22
```

### INDEX 写入规则

- 每个写操作（capture / weekly plan / weekly review / rename / archive / 状态变更 / 完成 / 放弃 / progress-update）结束时**必须**：
  - 刷新 `last_updated`
  - `version += 1`
  - 如果改变了 due_week / tier / 任务完成状态 / active 总数 → 刷新 Tasks Pool 概览计数
- weekly plan 结束时刷新 `last_weekly_plan`
- weekly plan 时刷新 `energy_this_week`（用户回答精力状态后写入：high / normal / low）
- weekly plan 时刷新 `work_hours_this_week`（用户回答本周全职工时后写入，v1.2.5 新增）
- weekly review 结束时刷新 `last_full_rebuild` 并完整重建 INDEX
- Tasks Pool 概览中的统计数据（active 总数、by_project 等）在每次刷新 INDEX 时从 active.md JSONL 行直接计算

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
- "滞留任务" = active.md 中 `due_week != null` 且 `due_week < current_week`（**比较时把 ISO 周转成日期，避免字符串字典序歧义**）且未完成

### current_week 切换时机

- 周复盘时：当周快照写入 review 文件**永久存档**，**不立即切换 current_week**
- 启动行为时：比对 INDEX.current_week 与今天 ISO 周；不一致 → 先自动保存上周最小快照，再主动询问用户"要不要现在开周计划？"
  - 用户同意 → 进入 weekly plan 工作流
  - 用户拒绝 → AI 自动**只更新 INDEX.current_week 为今天 ISO 周**，刷新 Tasks Pool 概览计数，整个修正走 journal

---

## 八、`.journal.md` 统一日志格式（v1.1 重新设计）

### 路径
- 当月：`${TASKOS_ROOT}/.journal.md`
- 历史：`${TASKOS_ROOT}/.journal-YYYY-MM.md`

### 格式：单行日志

```markdown
[2026-05-10 05:18] #287 done | complete t-20260510-003 → done-2026-05
[2026-05-10 05:19] #288 done | capture t-20260510-004 → active
[2026-05-10 06:00] #289 align | 坏行修复 active.md line 23
[2026-05-10 07:30] #290 decision | drop project「雅思7.5」reason: 优先级调整，精力不足
[2026-05-10 08:00] #291 in_progress | weekly-plan W20
[2026-05-10 08:15] #291 done | weekly-plan W20
```

### 标记类型

| 标记 | 含义 | 用途 |
|------|------|------|
| `done` | 常规写操作已完成 | 大多数单步操作 |
| `in_progress` | 多步操作开始 | 崩溃恢复检测；完成后写同 op 编号的 done |
| `align` | 数据修复/手工干预/改名记录 | 原 _align-log 的全部职责 |
| `decision` | 关键决策记录 | 复盘时可追溯"为什么" |
| `nudge` | AI 主动建议记录 | 建议内容 + 用户响应（采纳/忽略/拒绝） |
| `strategy` | Strategy 工作流操作 | 路线图创建/检视/调整/资源研究 |
| `gatekeeper` | 项目门禁判断记录 | 项目准入/拒绝 + 理由（v1.2.2） |
| `gatekeeper-override` | 用户强制覆盖门禁 | 用户坚持写入被拒项目（v1.2.2） |
| `nudge-韧性-拒绝` | 韧性检测被拒绝 | 冷却 2 周内不再提醒（v1.2.3） |
| `nudge-留白` | 探索空间提醒已发出 | 冷却 4 周内不再提醒（v1.2.3） |
| `profile-workload-update` | AI 自动更新了 profile 工作量基线 | 月度校准或用户告知全职变化时（v1.2.5） |

### 崩溃恢复规则

- 启动时读 `.journal.md` 末尾
- 如果最后一条 `in_progress` 没有对应的 `done`（同一 op 编号）→ 告知用户
- 处理选项：A 继续 / B 回滚 / C 标记已完成

### 月度归档

- 每次启动时检查 `.journal.md` 中最早一条记录的月份
- 早于当月 → 把所有早于当月的内容切到 `.journal-YYYY-MM.md`
- 切片粒度：按记录的时间戳决定归属

### 决策记录触发时机

AI 在以下操作时自动写一条 `[decision]`：
- drop 一个项目（含 reason）
- 升降 priority（含 reason）
- 调整 deadline（含 reason）
- 将 area status 改为 dormant/retired（含 reason）
- 用户在周计划中放弃某任务且给出了理由

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
- YAML 键名：`type`, `name`, `area`, `priority`, `deadline`, `status`, `progress`, `risk`, `key_milestones`, `yearly_intent`, `rituals`, `identity`, `created`, `milestone`, `desc`, `freq`, `energy_this_week`, `weekly_est_limit`, `weekly_est_limit_source`, `work_hours_this_week`, `work_hours`, `energy`, `data_schema`, `justification`
- YAML 枚举值：`active / dormant / retired`、`core / normal / side`、`active / paused / done / dropped`、`pending / done`、`weekly / monthly`、`high / normal / low`（energy）、`auto / manual`（weekly_est_limit_source）
- JSONL 字段名：`id / title / projects / area / context / est / carry / due_week / tier / captured / completed / outcome / week / ritual_source / ritual_desc / freq`
- JSONL 字段枚举值：`must / should / could`（tier）、`done / dropped`（outcome）、`weekly / monthly`（freq）
- ID 格式、ISO 周编号、TASKOS_ROOT 变量名、skill 自身文件名

### 用户自由
- area / project 名称：中文优先
- title / context / milestone / identity / yearly_intent / rituals.desc：自由书写

---

## 十一、周快照文件规范（v1.1 重新设计）

### 文件命名
`reviews/YYYY-Www.md`（如 `reviews/2026-W19.md`）

### frontmatter

```yaml
---
type: weekly_snapshot
week: 2026-W19
range: 2026-05-12 ~ 2026-05-18
generated: 2026-05-18
energy: normal                         # high / normal / low（从 INDEX.energy_this_week 取值）
work_hours: 42                         # v1.2.5 新增：本周全职工时（从 INDEX.work_hours_this_week 取值）
---
```

### 正文结构（纯结构化数据，无自由文本）

```markdown
# 2026-W19 周快照

## 项目状态
- 精读《动力取向心理治疗》: {progress: 0.50, risk: 🟢, priority: core}
- 播客第5期: {progress: 0.30, risk: 🟡, priority: core}
- 雅思7.5: {progress: 0.20, risk: 🟢, priority: core}
- 副项目X: {progress: 0.60, risk: 🟢, priority: normal}

## 本周数据
- planned: 10
- completed: 8
- dropped: 1
- carry_out: 2
- est_total: 9h

## 上下文分布
- @阅读: 5
- @写作: 2
- @电脑: 1
- @电话: 0

## carry 积压
- total_carry_tasks: 12
- carry_ge_3: 2

## 当周任务快照
### must
- [x] t-20260510-001 精读第 3 章
- [ ] t-20260511-002 联系督导
### should
- [x] r-2026-W19-001 精读 1 篇外刊
- [ ] t-20260512-003 写播客大纲
### could
- [ ] t-20260513-004 整理书架
```

### 字段说明

| 字段 | 含义 |
|------|------|
| `planned` | 本周排期的任务总数（due_week == 本周的非 ritual + ritual） |
| `completed` | 本周完成的任务数（done-*.md 中 week == 本周 且 outcome == done） |
| `dropped` | 本周放弃的任务数（done-*.md 中 week == 本周 且 outcome == dropped） |
| `carry_out` | 本周排了但未完成将 carry 到下周的任务数（due_week == 本周 且未完成 且非 ritual） |
| `est_total` | 本周排期任务的总估时（有 est 的累加，无 est 的按 1h 估算） |
| `total_carry_tasks` | active.md 中所有 carry > 0 的任务数 |
| `carry_ge_3` | active.md 中 carry ≥ 3 的任务数 |

### 向后兼容
- 旧格式的 review 文件（含 frontmatter `type: review`）AI 仍可正确读取
- 新写的 review 文件用 `type: weekly_snapshot` 格式
- 读快照时 AI 根据 frontmatter type 判断格式

---

## 十二、用户画像（profile.md）规范（v1.2 新增）

### 文件路径
`${TASKOS_ROOT}/profile.md`

### frontmatter

```yaml
---
type: user_profile
created: 2026-05-11
last_updated: 2026-05-11
completeness: 0.3                        # 0~1，已填写维度占比
---
```

### 正文结构（8 大维度）

```markdown
# 用户画像

## 基础信息
- 年龄段：
- 职业：
- 所在城市：
- 当前人生阶段：
- 可支配时间（工作日）：
- 可支配时间（周末）：

## 性格与认知风格
- 大五人格倾向：
  - 开放性：
  - 尽责性：
  - 外向性：
  - 宜人性：
  - 情绪稳定性：
- 决策风格：（直觉型/分析型/混合）
- 面对压力的模式：（回避/迎战/拖延后爆发/其他）
- 完美主义倾向：（高/中/低）

## 学习风格
- 偏好输入方式：（排序：阅读/视频/动手/听）
- 注意力持续时长：（典型深度工作 xx 分钟）
- 记忆方式偏好：（理解记忆/重复记忆/联想记忆）
- 新知识获取偏好：（体系化学习 vs 碎片化吸收）

## 精力与节奏
- 高效时段：
- 低谷时段：
- 周内节奏：（哪几天更容易产出）
- 周末偏好：（安排任务/完全放松/灵活）
- 精力恢复方式：

## 执行模式
- 任务粒度偏好：（大块深度 vs 小步快跑）
- 启动障碍：（什么情况容易拖延）
- 动力来源：（外部 deadline/内在兴趣/社交承诺/成就感/其他）
- 中断后恢复能力：（容易/困难）
- 惯用 context 偏好：

## 沟通偏好
- 激励方式：（具体数据/情感支持/对比进步）
- 批评接受度：（直接指出/温和引导/先扬后抑）
- 信息密度偏好：（精简/详细）
- 决策辅助偏好：（给选项/给建议/给分析让我自己想）

## 价值与优先级
- 当前人生阶段最重要的事（3~5 件）：
- 长期愿景（5~10 年）：
- 不可妥协的底线：

## 历史观察（AI 逐步积累，需用户确认后写入）
- 完成率规律：
- 拒绝建议的模式：
- 情绪波动关联因素：
- 擅长的领域：
- 容易卡住的类型：
```

### 冷启动规则

- `profile.md` 不存在 或 `completeness < 0.3` → 触发"认识你"对话
- 对话分 2~3 次自然穿插，每次 5~6 题，像朋友聊天
- 用户可跳过任何问题
- 每次写入后重算 completeness（已填维度数 / 总维度数）

### 更新规则

| 场景 | 行为 |
|------|------|
| 用户主动告知 | 立即写入对应段，刷新 last_updated |
| AI 发现数据规律 | 先问用户确认，确认后写入「历史观察」段 |
| Strategy 检视时 | 顺带问 1~2 个画像补充问题 |
| 用户显式修正 | 直接覆盖旧值，刷新 last_updated |

### 禁止行为

- ❌ 不做静默推断写入（必须问用户）
- ❌ 不在每次会话都分析性格
- ❌ 不将画像信息用于用户未授权的场景

### 工作量基线段（v1.2.5 新增）

profile.md 正文中新增一个顶级段落「工作量基线」，由 AI 自动维护（无需用户确认）：

```markdown
## 工作量基线（AI 自动维护，每月更新一次）

### 外部参考
- 深度工作日上限: 4h/天，20~28h/周（Ericsson 1993）
- 总认知工时最佳区间: 25~30h/周（Melbourne 2016）
- 全职后业余深度参考区间: _~_h/周（基于下方全职数据动态推导）

### 全职工作数据
- 典型周工作时长: _h（用户告知后 AI 写入，作为默认值）
- 工作认知强度: 高/中/低（高=全程需要深度思考；中=部分时段需要；低=大量机械性事务）
- 典型通勤时长: _h/天
- 本段最近更新: YYYY-MM-DD

### 个人历史指标（AI 从快照自动计算）
- 数据窗口: 近 N 周（最少 4 周有效数据后开始计算）
- 平均完成率: __%
- 甜点排期量: __h/周（完成率 ≥ 80% 的周的中位数 est_total）
- 各精力档完成率: high __% / normal __% / low __%
- 过载周占比: __%（carry_out/planned > 30% 的周数比例）
- 平均全职工时: __h/周（近 8 周用户报告值的均值）
- 长期趋势: 上升 / 稳定 / 下降（对比前 8 周）
- 本段最近更新: YYYY-MM-DD
```

**自动更新规则**：
- 每月首次启动：读最近 8 周快照 → 重算所有指标 → 写入
- 周复盘完成后（如果距上次更新 > 4 周）：同上
- 用户主动告知全职变化：立即更新全职工作数据段
- 写入后 journal 标记 `[profile-workload-update]`
- 此段**不计入 completeness 计算**（因为是 AI 自动维护，不属于用户画像维度）

---

## 十三、许愿卡奖励系统数据格式（v1.4.0 新增）

### 文件清单

| 文件 | 格式 |
|------|------|
| `rewards/ledger.md` | 标题 + JSONL 区块 |
| `rewards/bounties.md` | 标题 + JSONL 区块 |
| `rewards/wishlist.md` | Markdown checklist（自由格式） |
| `rewards/streaks.md` | 标题 + JSONL 区块 |
| `rewards/challenges.md` | 标题 + JSONL 区块 |

### Ledger 行格式

```json
{"id":"wc-YYYYMMDD-NNN","type":"earn|spend|adjust","amount":1,"reason":"xxx","source":"direct|bounty|streak|challenge","bounty_id":null,"streak_id":null,"challenge_id":null,"date":"YYYY-MM-DD"}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | `wc-YYYYMMDD-NNN` |
| `type` | string | ✅ | `"earn"` / `"spend"` / `"adjust"` |
| `amount` | integer | ✅ | 正整数 |
| `reason` | string | ✅ | 具体原因 |
| `source` | string | earn 时必填 | `"direct"` / `"bounty"` / `"streak"` / `"challenge"` |
| `bounty_id` | string | 否 | 关联悬赏 ID |
| `streak_id` | string | 否 | 关联连续纪录 ID |
| `challenge_id` | string | 否 | 关联挑战 ID |
| `date` | string | ✅ | YYYY-MM-DD |

### Bounties 行格式

```json
{"id":"b-YYYYMMDD-NNN","goal":"xxx","reward":2,"status":"active","auto_detect":null,"created":"YYYY-MM-DD","completed":null}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | `b-YYYYMMDD-NNN` |
| `goal` | string | ✅ | 目标描述 |
| `reward` | integer | ✅ | 完成后获得张数 |
| `status` | string | ✅ | `"active"` / `"completed"` / `"cancelled"` |
| `auto_detect` | string \| null | 否 | `"weekly_must_clear"` / `"project_done:项目名"` / `"streak:s-ID"` / null |
| `created` | string | ✅ | YYYY-MM-DD |
| `completed` | string \| null | 否 | 完成日期 |

### Streaks 行格式

```json
{"id":"s-YYYYMMDD-NNN","name":"xxx","freq":"daily","current_count":0,"best_count":0,"last_checkin":"YYYY-MM-DD","milestones":[7,14,30,60],"next_milestone":7,"status":"active","created":"YYYY-MM-DD"}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | `s-YYYYMMDD-NNN` |
| `name` | string | ✅ | 连续目标名称 |
| `freq` | string | ✅ | `"daily"` / `"weekly"` |
| `current_count` | integer | ✅ | 当前连续次数 |
| `best_count` | integer | ✅ | 历史最高 |
| `last_checkin` | string | ✅ | 上次打卡日期 |
| `milestones` | array | ✅ | 里程碑列表 |
| `next_milestone` | integer | ✅ | 下一个里程碑 |
| `status` | string | ✅ | `"active"` / `"paused"` / `"broken"` |
| `created` | string | ✅ | YYYY-MM-DD |

### Challenges 行格式

```json
{"id":"ch-YYYY-Www","challenge":"xxx","reward":2,"status":"pending","proposed":"YYYY-MM-DD","responded":null,"completed":null}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | `ch-YYYY-Www`（每周最多 1 条） |
| `challenge` | string | ✅ | 挑战描述 |
| `reward` | integer | ✅ | 完成奖励张数 |
| `status` | string | ✅ | `"pending"` / `"accepted"` / `"declined"` / `"completed"` / `"failed"` |
| `proposed` | string | ✅ | AI 提出日期 |
| `responded` | string \| null | 否 | 用户回应日期 |
| `completed` | string \| null | 否 | 完成日期 |

### INDEX.md Wish Cards 段

```yaml
## Wish Cards
wish_cards_balance: 0
wish_cards_earned_total: 0
wish_cards_spent_total: 0
active_streaks: 0
current_challenge: null
challenge_status: null
```

### ID 编码规范补充

| 类型 | 格式 | 示例 |
|------|------|------|
| 许愿卡流水 | `wc-YYYYMMDD-NNN` | `wc-20260529-001` |
| 悬赏 | `b-YYYYMMDD-NNN` | `b-20260520-001` |
| 连续纪录 | `s-YYYYMMDD-NNN` | `s-20260520-001` |
| 每周挑战 | `ch-YYYY-Www` | `ch-2026-W22` |

### JSONL 操作规则

与 tasks/active.md 相同的 JSONL 操作规则适用于所有 rewards/ 下的 JSONL 文件：
- 区块由 `## ~~~ JSONL 区块开始 ~~~` 和 `## ~~~ JSONL 区块结束 ~~~` 包围
- 每行合法 JSON，占满一行
- 修改用 id 定位 → 整行替换
- 坏行跳过 + journal [align]
