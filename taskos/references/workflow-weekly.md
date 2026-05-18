# Workflow: Weekly Plan + Weekly Review（周计划 + 周复盘）

负责本周排期、滞留任务处理、ritual 自动生成、风险评估、项目体检、core 标签复审、精力管理、工作量监控。

---

## 一、Weekly Plan（周计划）

### 触发词
- 开周计划 / 这周要做什么 / 排一下本周

### 流程概览（按顺序执行）

```
1. 懒人启动检测
2. 精力状态询问
3. 强制清 inbox
4. 滞留任务处理
5. Ritual 任务自动生成
6. 项目体检（风险模型）
7. 排期：必须做 / 该做 / 可以做 三档（根据精力动态调上限）
8. 工作量估算 + 留白提醒
9. 写回 active.md（修改 due_week + tier）
10. 重新生成 INDEX 当前周段
11. 刷新 last_weekly_plan + energy_this_week
12. journal [in_progress] → 整轮排期 → [done]
```

### 1.1 懒人启动检测

- 读 `INDEX.last_weekly_plan` 字段
- 用日期计算距今多久
- > 7 天 → 进入对齐对话：
  > 距上次开周计划已 X 天。建议先快速对齐当前状态：
  > - 「精读《...》」（core）：上次记录 progress=0.42，现在到了多少？
  > - 「播客第 5 期」（core）：上次记录 progress=0.18 + 🔴，要继续推还是调整？
  > - …
  > 请用一句话告诉我每个核心项目当前 milestone 完成度（0-100%）。

- 用户回答后，更新 project frontmatter 的 progress + 自动重算 risk

### 1.2 精力状态询问

AI 主动问：
> 这周整体精力状态怎样？
> - 充沛（可以多排点）
> - 正常
> - 低迷（少排为妙）

根据回答**动态调整三档上限**：

| 精力状态 | must ≤ | should ≤ | could ≤ |
|---|---|---|---|
| high（充沛） | 3 | 7 | 5 |
| normal（正常） | 2 | 5 | 3 |
| low（低迷） | 1 | 3 | 0 |

**低迷时 AI 主动建议**：
> 状态不好的时候，只做 must 档就够了。could 全跳过，should 也可以砍。给自己留点空白。

**记录位置**：精力状态写入 INDEX.energy_this_week 字段（high / normal / low），供周快照和复盘使用。

**边界情况**：如果用户跳过了 weekly plan 直接做 review，此时 energy_this_week 可能为空 → review 时 AI 先问"这周精力怎样？"再继续。

### 1.2b 韧性检测（连续高强度预警）

**数据来源**：读最近 2~3 个周快照文件（`reviews/YYYY-Www.md`）中记录的 energy_this_week 字段。快照不足 2 个时跳过此检测。

**触发逻辑**：

- 连续 2 周 high + 本周用户又选了 high：
  > 你连续 3 周都是"充沛"——挺好的，但也想提醒你：
  > 持续高转速容易感觉不到累，直到突然断电。
  > 这周要不要故意少排一档，给自己留个缓冲？

- 连续 3 周 high + 本周又 high：
  > ⚠️ 这是第四周高强度了。我认真建议这周按 normal 来排，
  > 把 could 档清空，留一个下午什么都不做。
  > 你当然可以拒绝，我只是你那个唠叨的朋友。

**行为约束**：
- 用户拒绝 → 不再追问，正常排期，journal 标记 [nudge-韧性-拒绝]
- 用户同意 → energy_this_week 降为 normal 档执行
- 冷却：拒绝后 2 周内不再重复，冷却记录写入 INDEX.md「Nudge 冷却」段（格式：`resilience_check: YYYY-Www`）

### 1.2c 本周全职工时采集（v1.2.5 新增）

**触发条件**：每次开周计划时必须执行。

AI 主动问：
> 这周工作大概多少小时？（加班/出差/调休都算在内）

**用户回答后**：
1. 记录为 `work_hours_this_week`
2. 写入 INDEX.md 的 `work_hours_this_week` 字段（与 energy_this_week 同级）
3. 结合研究基准动态计算本周业余深度参考上限（供 §1.8 使用）
4. 如果 work_hours_this_week > 52h：
   > ⚠️ 这周全职超过 52h 了。研究显示这个量级会实质性消耗认知资源。
   > 建议本周 TaskOS 任务大幅减排，只保留 must 档。

**用户跳过/不回答时**：
- 使用 profile.md「工作量基线 → 全职工作数据 → 典型周工作时长」作为默认值
- 该字段也无 → 使用 40h 作为保守默认值
- 不阻塞流程，继续下一步

**边界情况**：如果用户跳过了 weekly plan 直接做 review，此时 work_hours_this_week 可能为空 → review 生成快照时使用 profile 典型值。

### 1.3 强制清 inbox

- 列出 inbox 所有未处理条目
- 让用户**逐条**决定：
  - 挂哪个 project（→ 写 active.md，projects: [X]）
  - 仅挂 area（→ 写 active.md，projects: [], area: X）
  - 删除（→ 从 inbox.md 删除，不写 done.md）
- 处理完才进入下一步

### 1.4 滞留任务处理

**滞留任务定义**：active.md 中 `due_week != null && due_week < current_week` 且未完成的任务（不限上周；**比较时把 ISO 周转成日期**避免字符串歧义）。**不含 ritual 任务**。

列出来让用户**逐条**决定：

| 选项 | 操作 |
|---|---|
| **[继续做本周]** | due_week 改为 current_week，carry +1 |
| **[推迟]** | due_week 设为 null（回到任务池待排） |
| **[放弃]** | 在 active.md 删除 + 在 done-YYYY-MM.md append（outcome:dropped，completed=今天，week=任务原 due_week 或空） |
| **[实际已做]** | 默认 completed=今天（用户可改），移到 done-YYYY-MM.md（outcome:done） |

**carry ≥3 自动告警**：
> "X 这条已结转 3 次，要不要：A 拆得更小 / B 直接删掉 / C 重新评估它真的重要吗？"

**进行中任务额外显示 note**：
滞留任务中 `status == "in_progress"` 的，列出时附带 note（让用户带着上下文决策）：
> 「背 200 单词」（carry 1，上周进度：180/200）
>   → [继续做本周] / [标记完成] / [推迟] / [放弃]

**status/note 在"继续做本周"时保留原值，不重置**。

### 1.5 Ritual 任务自动生成

#### 周频 ritual（freq: weekly）
- **上周未做**：直接从 active.md 删除（不进 done-*.md，不计 carry，语义"每周一次错过即过"）
- **本周生成**：遍历所有 area 的 `rituals` 字段中 `freq: weekly` 的项 → 生成新 ritual 任务：
  ```json
  {"id":"r-2026-W19-NNN","title":"<desc>","projects":[],"area":"<area名>","context":null,"est":null,"carry":0,"due_week":"2026-W19","tier":"should","ritual_source":"<area名>","ritual_desc":"<desc>","freq":"weekly","captured":"<今天>"}
  ```
  - context 和 est 由 AI 根据 desc 推断（如"精读 1 篇外刊"→ context: "@阅读"）
  - tier 默认 "should"（rituals 进该做档）

#### 月频 ritual（freq: monthly）
- **生成时机**：仅在该月**第一周的 weekly plan** 时生成
- **判断逻辑**：AI 检查当月是否已生成过该月频 ritual
  - 扫描 active.md + 当月 done-YYYY-MM.md 中是否存在 `id` 以 `r-YYYY-MM-` 开头且 `ritual_desc` 匹配
  - 已存在 → 不重复生成
  - 不存在 → 生成 `r-YYYY-MM-NNN`，tier 默认 "should"
- **未完成处理**：当月最后一周仍未完成 → 静默放弃（直接从 active.md 删除）

### 1.6 项目体检（风险评估）

对所有 active project 计算 risk：

```
expected_progress = (今天 - created) / (deadline - created)
actual_progress   = project.progress 字段
gap               = actual_progress - expected_progress

🟢 gap ≥ -0.10
🟡 -0.30 ≤ gap < -0.10
🔴 gap < -0.30，或剩余天数 < 7 且 actual < 0.7
```

**温水煮青蛙检测**（仅当填了 key_milestones）：
- 剩余天数 < 50% 但 key_milestones 全 pending → 强制 🔴
- 剩余天数 < 25% 但 key_milestones 完成 < 50% → 强制 🔴

**优先级加权**：
- core 项目 🟡 → 处理时视为 🔴
- side 项目 🔴 → 仅在 weekly review 时提示，**weekly plan 时不强制**

写回 project frontmatter 的 risk 字段。

### 1.7 排期规则（三档）

| 档位 | 上限（默认/精力动态调整见 1.2） | 来源 |
|---|---|---|
| 必须做（must） | ≤ 3（精力调整后见上表） | core 项目关键任务 + 任何 🔴 normal 项目 |
| 该做（should） | ≤ 7（精力调整后见上表） | core 延伸任务 + normal 常规任务 + 本周新生成的 ritual |
| 可以做（could） | ≤ 5（精力调整后见上表） | side 项目任务 |

**排期来源**：
- 上一步处理后还在 active.md 中的、`due_week == null` 的任务
- 用户可以从滞留任务"继续做本周"的（已经被改为 current_week）
- 1.5 步骤新生成的 ritual

**操作**：修改 active.md 中相关任务的 `due_week` + `tier` 字段（用 id 定位 → 整行替换）。

**整轮排期作为单次 journal 事务**：[in_progress] → 执行 → [done]。

### 1.8 工作量估算 + 双锚点校准（v1.2.5 重写）

排期完成后，AI 自动执行：

**1. 计算本周 est_total**：
- 有 `est` 字段的任务：累加 est
- 无 `est` 字段的任务：按 1h 估算

**2. 计算当周外部参考上限**（基于 §1.2c 采集的 work_hours_this_week）：
- 工作日余量 = max(0, (35h - work_hours_this_week) / 5) × 5
- 周末余量 = 根据认知强度（profile.md）：高 4h / 中 6h / 低 7h
- research_ref = 工作日余量 + 周末余量
- 上限约束：research_ref ≤ 14h
- 降级：无 work_hours_this_week → 用 profile 典型值 → 再无 → 用 40h

**3. 读 profile.md「工作量基线 → 个人历史指标」**：
- personal_sweet_spot: 甜点排期量
- personal_completion_rate: 近 8 周平均完成率

**4. 推荐上限 = min(research_ref, personal_sweet_spot × 1.1)**：
- 历史数据不足 4 周 → 退化为仅用 research_ref
- personal_completion_rate < 70% → 推荐上限 = personal_sweet_spot × 0.9

**5. 对比 est_total vs 推荐上限**：
- a. est_total ≤ 推荐上限 → 不提醒，正常通过
- b. 超 10~20%：温和提醒
  > 本周排了约 Xh。你的历史甜点是 Yh（近 8 周完成率 ≥ 80% 的周平均值）。
  > 研究参考上限是 Zh（基于本周全职 Wh 推算）。建议砍 could 档 N 条。
- c. 超 20%+：认真建议
  > 本周排了约 Xh，超出你的历史最佳区间 Y%。
  > 过去排这么多的周，平均完成率只有 Z%。
  > 建议降到 Wh——全部做完的正反馈比多排几条重要。
- d. 连续 3 周 carry_out/planned > 30%：建议降 weekly_est_limit
  > 最近 N 周数据：[列出每周排期/完成/carry_out]
  > 建议把上限调到 Yh×1.1 ≈ Zh。

**6. 趋势提示**（如果 long_term_trend == "上升"，近 8 周 > 前 8 周 10%+）：
> 你最近 2 个月完成效率在持续提升。可以考虑把上限从 X 调到 Y。

**留白提醒**（每次排期结束都说）：
> "记得给自己留出什么都不做的时间——空白不是浪费，是恢复。"

### 1.8b 探索空间提醒

**数据来源**：读上一个周快照文件（`reviews/YYYY-Www.md`）中的排期档位填充情况。快照不存在时跳过此检测。

**触发条件**（同时满足）：
- 本周排期三档全满（must 达到上限 + should 达到上限 + could 达到上限）
- 上周快照显示也是三档全满

**提醒话术**：
> 连续两周都排得很满——你在 TaskOS 之外有没有留时间做点没目的的事？
> 散个步、看本闲书、随便逛逛都算。不用汇报给我。

**行为约束**：
- 不建议把"探索"写成 task（这恰恰是 Gatekeeper 该挡掉的）
- 不追踪、不打卡、不记录
- 这条提醒本质是"提醒你 TaskOS 之外还有生活"，不是"把生活也拉进 TaskOS"
- 每 4 周最多提醒 1 次，冷却记录写入 INDEX.md「Nudge 冷却」段（格式：`exploration_reminder: YYYY-Www`）
- journal 标记 [nudge-留白]

### 1.9 刷新 INDEX Tasks Pool 概览

排期完成后，AI 从 active.md 计算 Tasks Pool 概览各项计数并写入 INDEX。同时刷新：
- INDEX.last_updated
- INDEX.version += 1
- INDEX.last_weekly_plan = 今天日期
- INDEX.energy_this_week = 本次精力状态

计数项：
- active 总数：active.md JSONL 行数
- 本周排期：due_week == current_week 的任务数（分 must/should/could）
- 高 carry 任务：carry >= 3 的任务数
- 滞留任务：due_week < current_week 且未完成的任务数
- by_project：按 projects 字段聚合

### 1.10 Nudge 附加（v1.2 新增）

周计划生成完成后，如 INDEX.proactive.nudge == on：
- 检查是否有 strategy 子 project 在本周无任务排期
- 如有 → 温和提醒："xxx 路线图的「yyy」项目这周没排任务，需要加一条吗？"
- 不强制，用户可忽略
- 用户响应后走 journal [nudge]

---

## 二、Weekly Review（周复盘）

### 触发词
- 周复盘 / 本周怎么样

### 流程

```
1. 主动询问 core/normal active project 的 milestone 完成度
2. side 项目用户可跳过；跳过的 AI 自动重算 risk
3. 写回 progress + 重算 risk（单次 journal 事务）
4. 完整性扫描（4 项，问题记到 journal [align]）
5. INDEX 强制重建
6. 生成结构化周快照文件 reviews/YYYY-Www.md
7. core 标签复审
8. side 项目 🔴 提示
```

### 2.1 主动询问 progress

逐个 core / normal active project 询问：
> 「精读《动力取向心理治疗》」：上次记录 progress=0.42。本周 milestone 完成度到了多少？（0-100%）

### 2.2 side 项目跳过时的自动重算

公式：
- actual_progress 取 frontmatter 当前值（不变）
- expected_progress = (今天 - created) / (deadline - created)
- gap = actual - expected → 套等级表
- AI **写回 risk 字段，不更新 progress**（progress 是用户主观字段）

### 2.3 完整性扫描（3 项）

问题记到 `.journal.md`（[align] 标记）：

1. active.md 中 task 的 `projects` 数组任一元素引用了不存在的 project
2. project 引用了不存在的 area
3. active 目录有 project 但 INDEX 未列入

> ritual_source 不参与孤儿引用检查（rituals 字段被删时已生成的 ritual 任务靠 ritual_desc 自我描述，仍有效）。

### 2.4 软提示（🟡）

问题记到 `.journal.md`（[align] 标记）：

- active project 没挂 area
- active project 超 deadline 仍未关闭
- area 下无任何 active project
- inbox 累计 > 20 条
- side 项目处于 🔴

### 2.5 INDEX 强制重建

末尾从所有源文件**完整重建** INDEX，刷新 `last_full_rebuild`。

### 2.6 生成结构化周快照

写入 `reviews/YYYY-Www.md`（格式详见 schema.md 第十一节）：

```yaml
---
type: weekly_snapshot
week: 2026-W19
range: 2026-05-12 ~ 2026-05-18
generated: 2026-05-18
energy: normal
work_hours: 42
---
```

```markdown
# 2026-W19 周快照

## 项目状态
- 精读《动力取向心理治疗》: {progress: 0.50, risk: 🟢, priority: core}
- 播客第5期: {progress: 0.30, risk: 🟡, priority: core}

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

## carry 积压
- total_carry_tasks: 12
- carry_ge_3: 2

## 当周任务快照
### must
- [x] t-20260510-001 精读第 3 章
- [~] t-20260512-001 背 200 单词 | 180/200
### should
- [x] r-2026-W19-001 精读 1 篇外刊
- [ ] t-20260512-003 写播客大纲
### could
- [!] t-20260513-004 提交周报 | blocked: 等主管确认
- [ ] t-20260513-004 整理书架
```

**数据来源**：
- `项目状态` = 所有 active project 的 progress/risk/priority
- `planned` = active.md 中 due_week == current_week 的任务数
- `completed` = done-*.md 中 week == current_week 且 outcome == done
- `dropped` = done-*.md 中 week == current_week 且 outcome == dropped
- `carry_out` = active.md 中 due_week == current_week 且未完成 且非 ritual
- `est_total` = 排期任务 est 累加（无 est 按 1h）
- `上下文分布` = 排期任务按 context 字段聚合

**任务快照标记说明（v1.2.4 新增）**：
- `[x]` = 完成（在 done-*.md 中 outcome == done）
- `[~]` = 部分完成（status == "in_progress" 但到周末仍未完成，保留 note）
- `[ ]` = 未开始（status == "not_started" 或无 status 字段）
- `[!]` = 阻塞（status == "blocked"，保留 note）
- `carry 积压` = active.md 中 carry > 0 / carry ≥ 3 的计数

> **注意**：不切换 INDEX.current_week。current_week 只在下次 weekly plan 时切换（或启动行为时用户拒绝开 plan 自动修正）。

### 2.7 core 标签复审

> 当前 core 项目：[精读《...》, 播客第 5 期, 雅思 7.5]
> 这 3 个还是你最看重的吗？最近一个月有没有别的项目应该升 core？

如果用户想调整 → 通过 capture 的 priority 修改流程操作。

### 2.8 side 项目 🔴 提示

如果有 side 项目处于 🔴，单独列出来提示，但不强制处理。

### 2.9 Nudge 段（v1.2 新增）

- 如 INDEX.proactive.nudge == on
- 基于本周完成数据 + 下周排期状况，输出 ≤ 2 条建议
- 语气参考 profile.md 沟通偏好（如有）
- 用户响应后走 journal [nudge]

### 2.10 Strategy 关联检查（v1.2 新增）

- 如有 active strategy project
- 检查本周是否有关联子 project 的任务完成
- 简要关联到路线图当前阶段进度（如"词汇项目本周完成 3 条，阶段 1 整体 65%"）
- 纯信息展示，不做写操作

---

## 三、Weekly Plan + Weekly Review 之间的状态流转

```
周一开 plan
  ↓
排期 → INDEX 当前周快照生成 → last_weekly_plan 刷新
  ↓
（一周内）capture / 完成 / 改字段 / 修正 due_week …
  每次操作 → 走 journal → 刷 INDEX → 必要时重生成当周快照
  ↓
周日（或任意时间）开 review
  ↓
询问 progress → 重算 risk → 完整性扫描 → INDEX 强制重建
  ↓
生成结构化周快照存档 reviews/YYYY-Www.md
  ↓
core 复审 + side 🔴 提示
  ↓
（review 完成后 current_week 仍是本周）
  ↓
下周一（或下次启动）→ AI 检测到新周 → 自动保存最小快照 → 询问开周计划
```

---

## 四、AI 与用户对话模板（参考）

### Weekly Plan 启动（懒人模式）
> 距上次开周计划已 12 天。先对齐一下：
> - 精读《动力取向心理治疗》（core）：上次 progress=0.42。现在到了多少？
> - 播客第 5 期（core）：上次 0.18 + 🔴。还在推吗？
>
> 请用一句话告诉我每个核心项目的 milestone 完成度。

### 滞留任务处理
> 检测到 5 条滞留任务（due_week 已过但还没做完）：
>
> 1. t-20260428-003「整理第一章笔记」（精读 / 已结转 2 次）→ 继续做 / 推迟 / 放弃 / 已做？
> 2. ...

### Weekly Review 启动
> 开始本周复盘。逐个项目对齐 progress：
>
> 1. 精读《动力取向心理治疗》（core，上次 0.42）：本周到了多少？
> 2. ...
> 3. side 项目 X / Y / Z 你想跳过吗？跳过的话 AI 会基于时间流逝自动重算 risk。
