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

# 可选：关键里程碑（防温水煮青蛙）
key_milestones:
  - 完成全书精读: pending              # pending | done
  - 输出大纲: pending
---

（正文可选：详细规划、思路、参考资料等）
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
weekly_est_limit: 12h                  # 每周工作量软上限（用户可设）
data_schema: 1.1.0                     # v1.1 新增：当前数据 schema 版本
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

## Nudge 冷却（v1.2 新增）
- core_stall_「项目名」: 2026-W21
- carry_high: 2026-W22
```

### INDEX 写入规则

- 每个写操作（capture / weekly plan / weekly review / rename / archive / 状态变更 / 完成 / 放弃）结束时**必须**：
  - 刷新 `last_updated`
  - `version += 1`
  - 如果改变了 due_week / tier / 任务完成状态 → 重新生成"当前周"段
- weekly plan 结束时刷新 `last_weekly_plan`
- weekly plan 时刷新 `energy_this_week`（用户回答精力状态后写入：high / normal / low）
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
  - 用户拒绝 → AI 自动**只更新 INDEX.current_week 为今天 ISO 周**，并重新生成"当前周"段（基于 active.md 中 due_week == 新 current_week 的任务；可能为空），整个修正走 journal

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
- YAML 键名：`type`, `name`, `area`, `priority`, `deadline`, `status`, `progress`, `risk`, `key_milestones`, `yearly_intent`, `rituals`, `identity`, `created`, `milestone`, `desc`, `freq`, `energy_this_week`, `weekly_est_limit`, `energy`, `data_schema`
- YAML 枚举值：`active / dormant / retired`、`core / normal / side`、`active / paused / done / dropped`、`pending / done`、`weekly / monthly`、`high / normal / low`（energy）
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
