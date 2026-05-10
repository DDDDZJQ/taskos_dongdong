# Workflow: Weekly Plan + Weekly Review（周计划 + 周复盘）

负责本周排期、滞留任务处理、ritual 自动生成、风险评估、项目体检、core 标签复审。

---

## 一、Weekly Plan（周计划）

### 触发词
- 开周计划 / 这周要做什么 / 排一下本周

### 流程概览（按顺序执行）

```
1. 懒人启动检测
2. 强制清 inbox
3. 滞留任务处理
4. Ritual 任务自动生成
5. 项目体检（风险模型）
6. 排期：必须做 / 该做 / 可以做 三档
7. 写回 active.md（修改 due_week + tier）
8. 重新生成 INDEX 当前周段
9. 刷新 last_weekly_plan
10. journal [in_progress] → 整轮排期 → [done]
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

### 1.2 强制清 inbox

- 列出 inbox 所有未处理条目
- 让用户**逐条**决定：
  - 挂哪个 project（→ 写 active.md，projects: [X]）
  - 仅挂 area（→ 写 active.md，projects: [], area: X）
  - 删除（→ 从 inbox.md 删除，不写 done.md）
- 处理完才进入下一步

### 1.3 滞留任务处理

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

### 1.4 Ritual 任务自动生成

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

### 1.5 项目体检（风险评估）

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

### 1.6 排期规则（三档）

| 档位 | 上限 | 来源 |
|---|---|---|
| 必须做（must） | ≤ 3 | core 项目关键任务 + 任何 🔴 normal 项目 |
| 该做（should） | ≤ 7 | core 延伸任务 + normal 常规任务 + 本周新生成的 ritual |
| 可以做（could） | ≤ 5 | side 项目任务 |

**排期来源**：
- 上一步处理后还在 active.md 中的、`due_week == null` 的任务
- 用户可以从滞留任务"继续做本周"的（已经被改为 current_week）
- 1.4 步骤新生成的 ritual

**操作**：修改 active.md 中相关任务的 `due_week` + `tier` 字段（用 id 定位 → 整行替换）。

**整轮排期作为单次 journal 事务**：[in_progress] 列出所有计划改动 → 执行 → [done]。

### 1.7 重新生成 INDEX 当前周段

排期完成后，AI 从 active.md 过滤 `due_week == current_week` 的任务，按 tier 分组生成 INDEX 当前周快照。同时刷新：
- INDEX.last_updated
- INDEX.version += 1
- INDEX.last_weekly_plan = 今天日期

---

## 二、Weekly Review（周复盘）

### 触发词
- 周复盘 / 本周怎么样 / 复盘一下

### 流程

```
1. 主动询问 core/normal active project 的 milestone 完成度
2. side 项目用户可跳过；跳过的 AI 自动重算 risk
3. 写回 progress + 重算 risk + 更新 INDEX 当周快照（单次 journal 事务）
4. 完整性扫描 + INDEX 强制重建
5. 把当前 INDEX 当周快照复制写入 review 文件作为永久存档（不切换 current_week）
6. core 标签复审
7. side 项目 🔴 提示
8. 写 _align-log.md
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

### 2.3 完整性扫描（4 项）

写入 `reviews/_align-log.md`，🔴 级别：

1. active.md 中 task 的 `projects` 数组任一元素引用了不存在的 project
2. project 引用了不存在的 area
3. active 目录有 project 但 INDEX 未列入
4. **当周快照一致性**（按 ID 集合比较）：
   - 未完成段：INDEX 当周快照三档 ID 集合 == active.md 中 due_week == current_week 的 ID 集合
   - 已完成段：INDEX 当周快照"已完成本周" ID 集合 == done-YYYY-MM.md 中 week == current_week 且 outcome == "done" 的 ID 集合

> ritual_source 不参与孤儿引用检查（rituals 字段被删时已生成的 ritual 任务靠 ritual_desc 自我描述，仍有效）。

### 2.4 软提示（🟡）

写入 `reviews/_align-log.md`，🟡 级别：

- active project 没挂 area
- active project 超 deadline 仍未关闭
- area 下无任何 active project
- inbox 累计 > 20 条
- side 项目处于 🔴

### 2.5 INDEX 强制重建

末尾从所有源文件**完整重建** INDEX，刷新 `last_full_rebuild`。

### 2.6 当周快照存档

把当前 INDEX 当周快照段（含三档 + 已完成本周）**复制**写入 `reviews/2026-Www-review.md` 作为永久存档。

> **注意**：不切换 INDEX.current_week。current_week 只在下次 weekly plan 时切换（或启动行为时用户拒绝开 plan 自动修正）。

### 2.7 core 标签复审

> 当前 core 项目：[精读《...》, 播客第 5 期, 雅思 7.5]
> 这 3 个还是你最看重的吗？最近一个月有没有别的项目应该升 core？

如果用户想调整 → 通过 capture 的 priority 修改流程操作。

### 2.8 side 项目 🔴 提示

如果有 side 项目处于 🔴，单独列出来提示，但不强制处理。

### 2.9 review 文件结构

```markdown
---
type: review
week: 2026-W19
range: 2026-05-04 ~ 2026-05-10
generated: 2026-05-10
---

# 2026-W19 周复盘

## 项目进展
- 精读《动力取向心理治疗》（core）：progress 0.42 → 0.50（🟢）
- 播客第 5 期录制（core）：progress 0.18 → 0.30（🔴 → 🟡）
- ...

## 当周快照（永久存档）
（从 INDEX 复制的当周快照原文）

## 软提示与完整性扫描
（参见 _align-log.md 当周条目）

## core 复审结果
- 当前 core 不变 / 调整：X 升 core，Y 降 normal

## 备注
（用户自由记录的本周感受）
```

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
当周快照复制到 review 文件存档
  ↓
core 复审 + side 🔴 提示 + 写 _align-log
  ↓
（review 完成后 current_week 仍是本周）
  ↓
下周一（或下次 plan）→ AI 计算今天的 ISO 周 → 切换 current_week → 重新生成当周快照
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
