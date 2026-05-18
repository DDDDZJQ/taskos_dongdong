# Workflow: Weekly Plan + Weekly Review（周计划 + 周复盘）

---

## 一、Weekly Plan（周计划）

### 触发词
- 开周计划 / 这周要做什么 / 排一下本周

---

### ⚙️ 必须完成的动作（规则层，不可跳过）

无论 AI 选择什么交互方式，以下动作**必须全部完成**，顺序可灵活调整：

1. **清 inbox**：所有未处理项必须挂 project/area 或删除
2. **处理滞留任务**：due_week < current_week 的非 ritual 任务逐条决策（继续/推迟/放弃/已做）
3. **生成本周 ritual 任务**（按 area rituals 字段，详见下方规则）
4. **评估项目风险**：对所有 active project 计算 risk 并写回 frontmatter
5. **三档排期**（must / should / could）：修改 active.md 中任务的 due_week + tier
6. **INDEX 刷新**：last_updated, version, last_weekly_plan, energy_this_week, work_hours_this_week, Tasks Pool 概览计数
7. **journal 记录**：整轮排期作为单次事务（[in_progress] → ... → [done]）

---

### 💡 信息采集（指导层，AI 灵活采集）

开始排期前需要知道以下信息。**如果用户一句话给出了多个信息，不重复追问。**

| 信息 | 用途 | 如果缺失 |
|------|------|----------|
| 本周精力状态（high/normal/low） | 决定三档上限 | 主动问 |
| 本周预计全职工时（小时） | 计算业余深度余量 | 主动问；跳过则用 profile 默认值；再无则假设 40h |
| 距上次周计划是否 > 7 天 | 判断是否需要先对齐项目进度 | 从 INDEX.last_weekly_plan 自动读取 |

**精力档 → 排期上限参考**：

| 精力 | must ≤ | should ≤ | could ≤ |
|------|--------|----------|---------|
| high | 3 | 7 | 5 |
| normal | 2 | 5 | 3 |
| low | 1 | 3 | 0 |

**懒人对齐**：距上次周计划 > 7 天时，先主动引导用户对齐 core 项目进度（各项目 milestone 完成到多少了），更新 progress + 重算 risk 后再进入正式排期。

---

### 💡 主动检测与提醒（指导层，符合条件才说）

以下检测 AI 自行判断时机。**每次最多给 1~2 条主动建议，不该说的时候闭嘴。**

#### 韧性检测
- **数据来源**：近 2~3 个周快照的 energy 字段（不足 2 个时跳过）
- **意图**：连续多周 high 时提醒用户高强度不可持续
- **力度**：连续 2 周 high → 温和提一句；连续 3+ 周 → 认真建议降档
- **用户拒绝** → 不追问，journal [nudge-韧性-拒绝]，冷却 2 周

#### 工作量校准
- **时机**：排期完成后
- **意图**：确保排期量在用户实际能完成的范围内
- **参考锚点**：
  - 外部：有全职的人业余深度通常 6~12h/周（随工时和认知强度浮动）
  - 内部：近 8 周完成率 ≥ 80% 的周的中位数 est_total（甜点值）
- **判断原则**：
  - 在范围内 → 不说话
  - 超甜点 10~20% → 轻提一句
  - 超 20%+ 或连续多周 carry_out 过高 → 认真建议，引用具体数据
  - 趋势向好（近期完成率持续提升）→ 可以提示上调上限
- **不做**：不机械套公式然后报数字；不每次重复留白提醒；不在用户有动力时泼冷水

#### 探索空间提醒
- **条件**：连续 2 周三档全满
- **意图**：提醒 TaskOS 之外还有生活
- **频率**：每 4 周最多 1 次，冷却写入 INDEX Nudge 冷却段
- **不做**：不把"探索"写成 task，不追踪，不打卡

#### 超时预警
- **条件**：全职 > 52h
- **意图**：研究表明此量级影响脑健康（R3: 韩国 2025 脑结构研究）
- **行为**：建议大幅减排（只保留 must 档），不强制

#### Strategy 关联提醒
- **条件**：有 active strategy project 且其子 project 本周无排期
- **行为**：温和提醒，不强制

---

### 💡 阈值参考（指导层，AI 可灵活调整）

| 指标 | 建议初始值 | 调整原则 |
|------|-----------|----------|
| 甜点完成率阈值 | 80% | 大量周在 75~80% 且用户不焦虑时可下调 |
| 过载信号 | carry_out > 30% of planned | 任务粒度很小时比例自然高，酌情放宽 |
| 全职危险线 | 52h/周 | 有硬研究支撑，不建议调整 |
| 业余深度参考 | 6~12h/周 | 随实际全职工时浮动，工时越高区间越窄 |

AI 不需要每次都精确计算公式。理解意图（"别排超出能力太多"）比精确数字重要。

---

### ⚙️ Ritual 任务规则（规则层）

#### 周频 ritual（freq: weekly）
- 上周未做 → 从 active.md 删除（不进 done，不计 carry）
- 本周生成：遍历 area rituals 中 freq: weekly → 生成新行
  ```json
  {"id":"r-2026-W19-NNN","title":"<desc>","projects":[],"area":"<area名>","context":null,"est":null,"carry":0,"due_week":"2026-W19","tier":"should","ritual_source":"<area名>","ritual_desc":"<desc>","freq":"weekly","captured":"<今天>","status":"not_started","note":null}
  ```
- context 和 est 由 AI 根据 desc 推断

#### 月频 ritual（freq: monthly）
- 仅在该月第一周的 weekly plan 时生成
- 检查是否已存在：扫描 active.md + 当月 done-*.md
- 当月最后一周仍未完成 → 静默删除

---

### ⚙️ 风险模型（规则层）

```
expected_progress = (今天 - created) / (deadline - created)
actual_progress   = project.progress
gap               = actual_progress - expected_progress

🟢 gap ≥ -0.10
🟡 -0.30 ≤ gap < -0.10
🔴 gap < -0.30，或剩余天数 < 7 且 actual < 0.7
```

**温水煮青蛙**（仅当有 key_milestones）：
- 剩余天数 < 50% 但 key_milestones 全 pending → 强制 🔴
- 剩余天数 < 25% 但完成 < 50% → 强制 🔴

**优先级加权**：core 🟡 视为 🔴；side 🔴 仅 review 时提示。

---

### ⚙️ 排期规则（规则层）

| 档位 | 上限（由精力档决定） | 来源 |
|------|---------------------|------|
| must | 见上表 | core 关键任务 + 🔴 normal 项目 |
| should | 见上表 | core 延伸 + normal 常规 + 本周 ritual |
| could | 见上表 | side 项目任务 |

**排期来源**：due_week == null 的任务 + 滞留处理后"继续做本周"的 + 新生成 ritual

**操作**：修改 active.md 对应行的 due_week + tier（用 id 定位 → 整行替换）

---

### ⚙️ 滞留任务处理规则（规则层）

**定义**：active.md 中 `due_week != null && due_week < current_week` 且未完成。不含 ritual。比较时把 ISO 周转成日期。

**选项**：

| 选项 | 操作 |
|------|------|
| 继续做本周 | due_week → current_week，carry +1 |
| 推迟 | due_week → null |
| 放弃 | active 删行 + done append（outcome: dropped） |
| 实际已做 | done append（outcome: done），用户可改 completed 日期 |

- carry ≥ 3 时主动告警（提供拆分/删除/重评估选项）
- status == "in_progress" 的滞留任务列出时附带 note
- "继续做本周"时 status/note 保留原值

---

## 二、Weekly Review（周复盘）

### 触发词
- 周复盘 / 本周怎么样

---

### ⚙️ 必须完成的动作（规则层）

1. **询问项目进度**：core/normal 逐个问 progress；side 可跳过
2. **写回 progress + 重算 risk**（单次 journal 事务）
3. **完整性扫描**（3 项，问题记 journal [align]）：
   - task 引用了不存在的 project
   - project 引用了不存在的 area
   - active 目录有 project 但 INDEX 未列入
4. **INDEX 强制重建**（从源文件完整重建，刷新 last_full_rebuild）
5. **生成结构化周快照** `reviews/YYYY-Www.md`（格式详见 schema.md）
6. **journal 记录**

---

### 💡 Review 中的主动判断（指导层）

- **core 标签复审**：问用户当前 core 还是不是最看重的（可灵活判断是否需要问）
- **side 🔴 提示**：如果有 side 项目 🔴，单独提一下但不强制处理
- **Nudge 建议**：基于本周完成数据 ≤ 2 条建议
- **Strategy 关联**：如有 active strategy，简要展示本周子 project 进展
- **软提示**：project 没挂 area / 超 deadline 未关闭 / inbox > 20 条 等

---

### ⚙️ 周快照格式（规则层）

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

**正文包含**：项目状态 / 本周数据（planned, completed, dropped, carry_out, est_total）/ 上下文分布 / carry 积压 / 当周任务快照

**任务快照标记**：`[x]` 完成 / `[~]` 部分完成（in_progress 但未完成，保留 note）/ `[ ]` 未开始 / `[!]` 阻塞（保留 note）

**数据来源**：
- planned = due_week == current_week 的任务数
- completed = done-*.md 中 week == current_week 且 outcome == done
- dropped = done-*.md 中 week == current_week 且 outcome == dropped
- carry_out = due_week == current_week 且未完成 且非 ritual
- est_total = 排期任务 est 累加（无 est 按 1h）

> current_week 不在 review 时切换。只在下次 plan 或启动自动修正时切换。

---

## 三、语气指导（替代所有具体话术模板）

- 像朋友聊天，不像系统通知
- 提醒过载时引用具体数据（"你上 3 周平均完成 7 条，这次排了 12 条"），不引用公式名
- 韧性提醒承认"你可能真的精力旺盛"但也提供另一种可能，说一次就够
- 建议降档时给理由 + 给选项（"要砍哪些？"），不给命令
- 趋势向好时直接夸，不附加"但是..."
- 低迷时先共情再给建议
- 滞留任务 carry ≥ 3 时态度直接但不刻薄
