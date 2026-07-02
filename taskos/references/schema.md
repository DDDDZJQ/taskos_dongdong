# TaskOS 数据模型规范（schema）

本文件定义 TaskOS 的全部数据结构、文件格式、字段语义和操作规则。所有工作流的写操作都必须遵守本文件。

---

## 一、文件清单与格式映射

| 文件 | 格式 |
|---|---|
| `INDEX.md` | Markdown（含表格、列表） |
| `.journal.md` / `.journal-YYYY-MM.md` | 单行日志格式（append-only） |
| `areas/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/active/{中文名}.md` | Markdown + YAML frontmatter |
| `projects/archive/{中文名}.md` | 同上 |
| `tasks/inbox.md` | Markdown checklist |
| **`tasks/active.md`** | **标题 + JSONL 区块** |
| **`tasks/done-YYYY-MM.md`** | **标题 + JSONL 区块** |

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
justification: "简述为什么这个项目值得被 TaskOS 管理"  # 项目创建时写入

# 可选：关键里程碑
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
- ## 偏好
- ## 备注
```

**约束**：
- `priority: core` 全局同时存在数 ≤ 3（strategy project 不计入）
- `deadline - created` ≤ 90 天
- `status: done | dropped` 满 14 天自动归档到 `projects/archive/`

---

### Strategy Project frontmatter

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
---
```

**与普通 project 的差异**：
- `type: strategy`（普通 project 为 `project` 或无 type 字段）
- 使用 `goal` 替代 `milestone`
- 使用 `target_date` 替代 `deadline`，**不受 90 天约束**
- 新增 `last_reviewed` 字段（上次检视日期）
- **不占用 core ≤ 3 槽位**
- 进度公式：已完成阶段数 / 总阶段数

**子 project 新增字段**：

```yaml
parent_strategy: "[strategy] 两年达成雅思 7.5"
strategy_phase: 1
```

- `parent_strategy`：指向父 strategy project 的 name 字段
- `strategy_phase`：属于路线图第几阶段（从 1 开始）
- 子 project 本身是普通 project，正常参与 deadline 约束

---

## 四、Task — JSONL 行格式

### 4.1 active.md / done-YYYY-MM.md 整体结构

```markdown
# Active Tasks
## ~~~ JSONL 区块开始 ~~~
{"id":"t-20260510-001","title":"精读第 3 章","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@阅读","est":"90min","captured":"2026-05-10","status":"not_started","note":null}
{"id":"r-2026-W19-001","title":"精读 1 篇外刊封面文章","projects":[],"area":"心理学","context":"@阅读","est":"60min","ritual_source":"心理学","ritual_desc":"精读 1 篇外刊封面文章","freq":"weekly","captured":"2026-05-10","status":"not_started","note":null}
## ~~~ JSONL 区块结束 ~~~
```

> done-YYYY-MM.md 标题为 `# Done Tasks YYYY-MM`，其余结构相同。

### 4.2 active.md 任务字段定义

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `id` | string | ✅ | `t-YYYYMMDD-NNN` / `r-YYYY-Www-NNN` / `r-YYYY-MM-NNN` |
| `title` | string | ✅ | 任务描述（动词前置） |
| `projects` | array | ✅ | 关联项目数组（可空 []） |
| `area` | string \| null | 否 | 仅挂 area 不挂 project 时填 |
| `context` | string | 否 | 情境标签（@阅读 / @电脑 / @电话 / @外出 / @写作 …） |
| `est` | string | 否 | 估时（"90min" / "2h"） |
| `ritual_source` | string | 否 | 仪式型任务的来源 area 名 |
| `ritual_desc` | string | 否 | 仪式型任务的原始 desc 文本 |
| `freq` | string | 否 | 仪式型频率（"weekly" / "monthly"），仅 ritual 任务有 |
| `captured` | string | ✅ | 捕获日期（YYYY-MM-DD） |
| `status` | string | 否 | `"not_started"` / `"in_progress"` / `"blocked"`；默认 not_started |
| `note` | string \| null | 否 | 自由文本进度细节；每次更新覆盖旧值；旧行无此字段视为 null |

**v2.1.0 删除字段**：`due_week`、`tier`、`carry`（周周期排期机制已删除）

### 4.3 done-YYYY-MM.md 额外字段

| 字段 | 必填 | 说明 |
|---|---|---|
| `completed` | ✅ | 完成或放弃的日期（YYYY-MM-DD） |
| `outcome` | ✅ | "done" 或 "dropped" |
| `week` | 否 | 完成时所属 ISO 周 |

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
- 任务 captured 日期所在月的 `done-YYYY-MM.md`

NNN 从 001 起递增。

---

## 七、INDEX.md 结构

```markdown
# TaskOS Index
last_updated: 2026-05-10 05:18:00
version: 287
data_schema: 2.1.0                     # 当前数据 schema 版本
proactive:                             # 主动规划开关
  nudge: on                            # on | off
  strategy: on                         # on | off

## Areas (active: 4)
- 心理学 (core: 1, normal: 2, side: 0)
- 自媒体 (core: 1, normal: 0, side: 1)

## Projects — Core (3/3) ⚠️ 已满
| name | area | progress | deadline |
|---|---|---|---|
| 精读《...》 | 心理学 | 42% | 2026-08-10 |

## Projects — Normal (5)
（同上表格式）

## Projects — Side (2)
（同上表格式）

## Strategy Projects
| name | area | progress | target_date | last_reviewed |
|---|---|---|---|---|
| [strategy] 两年达成雅思 7.5 | 英语学习 | 25% | 2028-05 | 2026-08-15 |

## Tasks Pool 概览
- inbox: 7 条（最久 8 天）
- active 总数: 47
- by_project: 精读《...》12 / 播客 8 / _orphan 3 / _ritual 4

## Nudge 冷却
- core_stall_「项目名」: 2026-W21
```

### INDEX 写入规则

- 每个写操作（capture / rename / archive / 状态变更 / 完成 / 放弃 / progress-update / 统一复盘）结束时**必须**：
  - 刷新 `last_updated`
  - `version += 1`
  - 如果改变了任务完成状态 / active 总数 → 刷新 Tasks Pool 概览计数
- Tasks Pool 概览中的统计数据（active 总数、by_project 等）在每次刷新 INDEX 时从 active.md JSONL 行直接计算

### 启动校验

每次启动 skill 时：
1. 全量读 INDEX.md 加载到上下文
2. 轻量计数校验：
   - active.md JSONL 行数 vs INDEX `active 总数`
   - `areas/` 目录下 .md 文件数 vs INDEX 列出的 area 数
   - `projects/active/` 目录下 .md 文件数 vs INDEX **Core + Normal + Side + Strategy 四段**累加数
3. 任一不一致 → 提示用户"INDEX 可能漂移"，询问是否当场重建

### 滞留判断

由于 v2.1.0 删除了周周期排期机制，不再有基于 ISO 周的「滞留任务」概念。
判断任务是否长期未处理的标准：`created` 距今天 > 14 天且 `status` != "blocked"。

---

## 八、`.journal.md` 统一日志格式

### 路径
- 当月：`${TASKOS_ROOT}/.journal.md`
- 历史：`${TASKOS_ROOT}/.journal-YYYY-MM.md`

### 格式：单行日志

```markdown
[2026-05-10 05:18] #287 done | complete t-20260510-003 → done-2026-05
[2026-05-10 05:19] #288 done | capture t-20260510-004 → active
[2026-05-10 06:00] #289 align | 坏行修复 active.md line 23
[2026-05-10 07:30] #290 decision | drop project「雅思7.5」reason: 优先级调整，精力不足
[2026-05-10 08:00] #291 in_progress | review
[2026-05-10 08:15] #291 done | review 项目进度核对，更新 3 个 project progress
```

### 标记类型

| 标记 | 含义 | 用途 |
|------|------|------|
| `done` | 常规写操作已完成 | 大多数单步操作 |
| `in_progress` | 多步操作开始 | 崩溃恢复检测；完成后写同 op 编号的 done |
| `align` | 数据修复/手工干预/改名记录 | 原 _align-log 的全部职责 |
| `decision` | 关键决策记录 | 复盘时可追溯"为什么" |
| `nudge` | AI 主动建议记录 | 建议内容 + 用户响应 |
| `strategy` | Strategy 工作流操作 | 路线图创建/检视/调整 |
| `nudge-韧性-拒绝` | 韧性检测被拒绝 | 冷却期内不再提醒 |
| `nudge-留白` | 探索空间提醒已发出 | 冷却期内不再提醒 |
| `nudge-恢复-拒绝` | 恢复协议被拒绝 | 冷却期内不再提醒 |
| `profile-workload-update` | profile 工作量基线更新 | 复盘时手动触发 |

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
- 用户在复盘中放弃某任务且给出了理由

### 触发条件
**所有写操作都走 journal，无例外**。

---

## 九、语言约定

### 必须中文
- SOP 说明、决策树、提示语、注释
- 工作流标题、章节标题
- 错误提示 / 用户对话模板

### 必须英文（机器可读性）
- YAML 键名：`type`, `name`, `area`, `priority`, `deadline`, `status`, `progress`, `key_milestones`, `yearly_intent`, `rituals`, `identity`, `created`, `milestone`, `desc`, `freq`, `data_schema`, `justification`
- YAML 枚举值：`active / dormant / retired`、`core / normal / side`、`active / paused / done / dropped`、`pending / done`、`weekly / monthly`、`auto / manual`
- JSONL 字段名：`id / title / projects / area / context / est / captured / completed / outcome / week / ritual_source / ritual_desc / freq / status / note`
- JSONL 字段枚举值：`done / dropped`（outcome）、`weekly / monthly`（freq）、`not_started / in_progress / blocked`（status）
- ID 格式、ISO 周编号、TASKOS_ROOT 变量名、skill 自身文件名

### 用户自由
- area / project 名称：中文优先
- title / context / milestone / identity / yearly_intent / rituals.desc：自由书写

---

## 十、用户画像（profile.md）规范

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
| 统一复盘时 | 顺带问 1~2 个画像补充问题 |
| 用户显式修正 | 直接覆盖旧值，刷新 last_updated |

### 禁止行为

- ❌ 不做静默推断写入（必须问用户）
- ❌ 不在每次会话都分析性格
- ❌ 不将画像信息用于用户未授权的场景

### 工作量基线段（AI 自动维护）

profile.md 正文中新增一个顶级段落「工作量基线」，由 AI 自动维护（无需用户确认）：

```markdown
## 工作量基线（AI 自动维护，定期更新）

### 外部参考
- 深度工作日上限: 4h/天，20~28h/周（Ericsson 1993）
- 总认知工时最佳区间: 25~30h/周（Melbourne 2016）

### 全职工作数据
- 典型周工作时长: _h（用户告知后 AI 写入）
- 工作认知强度: 高/中/低
- 典型通勤时长: _h/天
- 本段最近更新: YYYY-MM-DD

### 个人历史指标（AI 从 done 文件 + project 进度推算）
- 数据窗口: 近 N 周（最少 4 周有效数据后开始计算）
- 平均完成率: __%
- 各精力档完成率: high __% / normal __% / low __%
- 长期趋势: 上升 / 稳定 / 下降
- 本段最近更新: YYYY-MM-DD
```

**自动更新规则**：
- 用户触发统一复盘时，AI 酌情更新
- 写入后 journal 标记 `[profile-workload-update]`
- 此段**不计入 completeness 计算**
