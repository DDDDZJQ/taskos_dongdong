# Workflow: Distill（月度自动蒸馏）

每月自动执行的总结性工作流，输出 `reviews/distill/YYYY-MM.md`，包含 area 活跃度、yearly_intent 进度、行动力洞察等。**自动触发，不需要用户主动调用**。

---

## 一、触发时机

由 SKILL.md 启动行为自动检查：
- 每月 1 日（或下次启动时检测到 `last_updated` 早于本月）→ 自动跑上月的 distill
- 检查依据：`INDEX.md` 中的 `## 上次月度蒸馏` 字段
- 已为本月（或更晚）跑过 → 跳过

---

## 二、输出文件结构

`reviews/distill/YYYY-MM.md`：

```markdown
---
type: distill
month: 2026-04
generated: 2026-05-01
---

# 2026-04 月度蒸馏

## 一、各 area 完成项目数 / 完成率

| area | 月初 active | 月内新建 | 月内完成 | 月内放弃 | 月末 active | 完成率 |
|---|---|---|---|---|---|---|
| 心理学 | 3 | 1 | 1 | 0 | 3 | 25% |
| ...

> rename 历史合并：本月有「X」改名为「Y」的记录，统计时视为同一对象。

## 二、本月已归档项目一行总结

- 「精读《XX》」（心理学，core）：4 月 28 日完成，progress 1.00 / risk 🟢。耗时 78 天，按时交付。
- 「Y」（自媒体，normal）：4 月 15 日 dropped。原因（如有 _align-log 记录）：…
- ...

## 三、行动力洞察

- 本月 task 完成数：47 条（done）+ 8 条（dropped）
- @阅读 占比 35%，@电脑 25%，@写作 20%，其他 20%
- 高 carry 任务（≥3）：3 条，全部集中在「Y 项目」
- 滞留模式：每月初容易堆积滞留任务（建议每周开 plan）
- 完成时段分布：（基于 done-*.md 的 completed 日期）

## 四、area 活跃度排行

| area | 完成 task 数 | 新建 project 数 | 完成 project 数 |
|---|---|---|---|
| 心理学 | 22 | 1 | 1 |
| 自媒体 | 12 | 0 | 0 |
| 跨文化 | 8 | 0 | 0 |
| 运动 | 5 | 0 | 0 |

## 五、area 活跃度警报

- ⚠️ area「运动」连续 3 个月 0 active project + 完成 task < 5 → 建议改 status: dormant
  （需用户确认；建议下次启动时询问）

## 六、yearly_intent 进度回顾

- 心理学：yearly_intent = "今年完成长程培训第二年学习 + 输出 3 篇深度综述"
  - 至今进度感：长程培训按计划，综述已完成 1 篇，还差 2 篇
  - 距年底 8 个月：需要每 4 个月输出 1 篇，节奏可控
  - AI 判断：🟢 按节奏

- 自媒体：yearly_intent = "..."
  - ...

## 七、本月主要决策与改名记录

（从 _align-log.md 提取本月改名记录）

- 2026-04-15：area「X」→「Y」

## 八、给用户的反思提示

- 本月哪些 core 项目持续推进？哪些卡住？
- yearly_intent 是否还合适？是否需要在 5 月调整？
- 哪个 area 应该升降优先级？

## 九、精力分布与工作效率（v1.0 新增）

### 本月各周精力与产出对照
| 周 | 精力 | 完成量 | est 总计 | 备注 |
|---|------|--------|---------|------|
| W18 | 正常 | 12 | 10h | |
| W19 | 低迷 | 8 | 6h | 状态欠佳，正常回落 |
| W20 | 充沛 | 15 | 13h | 高效周 |
| W21 | 正常 | 10 | 9h | |

### 精力趋势（保留时间线）
- 本月：充沛 1 周 / 正常 2 周 / 低迷 1 周
- 上月：充沛 2 周 / 正常 1 周 / 低迷 1 周
- 两月对比：低迷周占比持平（25%）

### 洞察
- 低迷周平均完成量 = 8 条，充沛周 = 15 条（差异正常，不必焦虑）
- 连续低迷未出现（健康信号）
```

> 数据来源：各周 review 文件的 frontmatter.energy 字段 + done-*.md 按周统计。

---

## 三、执行流程

### 步骤 1：journal [in_progress]

```markdown
## 2026-05-01 02:00:00 [in_progress] op=#NNN
intent: 执行 2026-04 月度蒸馏
files_planned:
  - reviews/distill/2026-04.md（新建）
  - INDEX.md（"上次月度蒸馏"字段刷新为 2026-04 / 执行于 2026-05-01）
```

### 步骤 2：扫描数据源

读取以下文件以汇总数据：
- `areas/*.md`（含 archive？通常 distill 只看 active 和当月转 dormant 的）
- `projects/active/*.md` + `projects/archive/*.md`（识别本月归档的）
- `tasks/done-2026-04.md`（本月完成 / 放弃的 task）
- `tasks/active.md`（月末快照：剩余任务）
- `reviews/_align-log.md`（本月改名记录）
- `reviews/2026-W14-review.md` ~ `2026-W17-review.md`（本月所有周复盘，含 energy 字段）

### 步骤 3：聚合统计

#### 3.1 area 活跃度
- 完成 task 数 = `done-2026-04.md` 中 outcome:done 且 area 字段 == X 或 projects 数组中任一 project 的 area == X
- 新建 project 数 = projects/ 中 created 在本月内的
- 完成 project 数 = projects/archive/ 中 status: done 且归档日期在本月内的

#### 3.2 area 活跃度警报
- 检查每个 area，看连续 3 个月（含本月）的活跃度
- 完成 task < 5 且新建/完成 project = 0 → 触发警报建议 dormant

#### 3.3 行动力洞察
- @context 分布
- 高 carry 任务集中在哪些 project
- 滞留模式
- 完成时段分布（基于 completed 日期的星期 / 时段）

#### 3.4 yearly_intent 进度
- 对每个填了 yearly_intent 的 area：
  - AI 基于本月 + 累积完成的 project + milestone 文本，给出"进度感"判断
  - 计算"距年底剩余月数 vs 剩余目标"
  - 给 🟢 / 🟡 / 🔴 判断

#### 3.5 精力分布统计（v1.0 新增）
- 读取本月各周 review 文件的 frontmatter.energy 字段
- 按周列出精力状态 + 对应完成量（从 done-*.md 按 week 字段统计）
- 计算 est 总计（有 est 的累加，无 est 的按 1h 估算）
- 对比上月精力分布
- 给出精力-产出关联洞察

### 步骤 4：rename 历史合并

读 `_align-log.md` 中所有改名记录，统计时把改名前后的同一对象视为一个：
- 例如「X 在 2026-03-10 改名为 Y」→ 4 月统计时 X 和 Y 的所有 task / project 都算给 Y

### 步骤 5：写文件 + 更新 INDEX

- 写 `reviews/distill/YYYY-MM.md`
- 更新 INDEX.md 的"上次月度蒸馏"字段："YYYY-MM（执行于 ZZZZ-MM-DD）"
- INDEX.last_updated / version += 1

### 步骤 6：journal [done] + Mini-Check

---

## 四、用户反馈点

distill 跑完后，AI 在与用户的下一次对话中**主动提示**：

> 4 月月度蒸馏已生成。要不要看一下？里面有：
> - 「运动」area 连续 3 个月零活跃 → 建议改 dormant
> - 心理学 yearly_intent 节奏可控 🟢
> - 高 carry 任务集中在「Y 项目」可能要重新评估
>
> （路径：reviews/distill/2026-04.md）

如果用户没回应，下次启动时不再重复提示（避免烦人）。

---

## 五、回填 / 漏跑处理

如果用户长期不开 skill，回来时发现已经过了多个月没跑 distill：
- AI 不要一次性跑所有缺失月份（噪声大、价值低）
- 只跑**最近一个月**的 distill
- 提示用户："已跑 2026-09 月度蒸馏。中间 5/6/7/8 月未跑，是否需要补？"

如果用户说"补 5-8 月"→ 逐月跑；如果说"算了"→ 跳过，但在 _align-log.md 留一笔"5-8 月 distill 跳过"。

---

## 六、Mini-Check（distill 后）

| 检查项 | 说明 |
|---|---|
| reviews/distill/YYYY-MM.md 已生成 | 文件存在 + 含必要章节 |
| INDEX 上次月度蒸馏字段更新 | 反映最新执行月份和日期 |
| dormant 建议是否需要用户确认 | 如有 → 下次启动时主动询问 |

---

## 七、对话模板（参考）

### distill 完成后通知
> 已完成 4 月月度蒸馏（reviews/distill/2026-04.md）。
>
> 关键点：
> - 「运动」area 连续 3 个月零活跃 → 建议改 dormant，要现在改吗？
> - yearly_intent 检查：心理学 🟢，自媒体 🟡（差 1 期播客），跨文化 🟢
> - 行动力洞察：@阅读 占 35%，与 yearly_intent 节奏一致

### 用户问"我这个月做了什么"
> 看下 reviews/distill/2026-04.md。简要：
> - 完成 22 条 task（done），放弃 5 条
> - 1 个 project 完成（精读《XX》）
> - …
