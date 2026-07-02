# TaskOS 迁移指引

当 skill 版本升级后数据格式可能发生变化，本文件记录每个版本的迁移动作。**由用户手动触发**，不在启动时自动执行。

---

## 一、触发词

- 执行迁移 / 升级数据 / migrate / 数据迁移

---

## 二、执行流程

1. 读 INDEX.md 的 `data_schema` 字段（不存在视为 `1.0.0`）
2. 对比当前 SKILL.md 的 `version` 字段
3. 如果 SKILL version > data_schema → 列出需要执行的迁移步骤
4. **向用户展示迁移内容，等待确认后才执行**
5. 用户确认后逐版本执行迁移动作
6. 完成后更新 INDEX.data_schema
7. 走 journal 记录

---

## 三、关键原则

1. **用户确认后才执行**：先列出所有变更，用户说"执行"后才动手
2. **幂等性**：每个迁移步骤都有前置检查，重复执行不会出错
3. **向后兼容**：旧格式文件保持不动，AI 在读取时自适应判断格式
4. **走 journal**：实际文件变更时走 journal（[in_progress] → 逐步执行 → [done]）

---

## 四、迁移记录

### v1.1.0（从 v1.0.0 升级）

#### 新增

- INDEX.md 新增 `data_schema: 1.1.0` 字段
- review 文件改为结构化快照格式（`type: weekly_snapshot`）
- journal 新增 `[decision]` 和 `[align]` 标记
- 自动最小快照机制（current_week 切换时保存）

#### 废弃

- `reviews/_align-log.md` → 内容转写到 .journal.md
- `reviews/distill/` 目录 → 删除（按需生成不再预存）
- `active.md` 顶部元数据（last_updated/total/by_project_count）→ 删除
- `done-*.md` 顶部元数据 → 删除
- review 文件后缀 `-review` → 改为无后缀（旧文件保留不动）

#### 一次性迁移动作

```
1. 在 INDEX.md 元数据区添加 `data_schema: 1.1.0`

2. 去掉 INDEX.md 的 `## 上次月度蒸馏` 段（含其下内容）

3. 如果 `reviews/_align-log.md` 存在：
   - 读取其中所有条目
   - 按条转写到 .journal.md 末尾（格式：[原始日期 00:00] #NNN align | 内容）
   - op 编号从当前 journal 最大编号 +1 递增
   - 删除 `reviews/_align-log.md`

4. 如果 `reviews/distill/` 目录存在：
   - 删除该目录及其下所有文件

5. active.md 格式简化：
   - 删除 `# Active Tasks` 标题行与 `## ~~~ JSONL 区块开始 ~~~` 之间的所有行
   - 保留标题行和 JSONL 区块（结果：标题行紧接 JSONL 开始标记）

6. done-*.md 格式简化（对 tasks/ 下所有 done-*.md 执行）：
   - 同上，删除标题行与 JSONL 区块开始之间的元数据行

7. 已有的 review 文件（reviews/*-review.md）保持不动
   - 不重命名、不修改格式
   - AI 读取时根据 frontmatter type 判断格式（type: review = 旧格式）
```

#### 行为变更（迁移后自动生效，无需额外操作）

- journal 写入改用单行格式
- 写操作后不再重算 active.md/done-*.md 顶部统计
- 月度蒸馏不再自动触发，改为用户请求"回顾"时实时生成
- current_week 切换时自动保存最小周快照
- 改名记录写到 journal `[align]` 而非 _align-log.md
- 完整性扫描问题写到 journal `[align]` 而非 _align-log.md

#### 验证清单

迁移完成后 AI 自检：

| 检查项 | 预期 |
|--------|------|
| INDEX.data_schema | == "1.1.0" |
| _align-log.md | 已删除或不存在 |
| reviews/distill/ | 目录已删除或不存在 |
| active.md 格式 | 标题行下直接是 JSONL 区块开始标记 |
| done-*.md 格式 | 同上 |
| INDEX 无"上次月度蒸馏"段 | 确认已删除 |

全部通过 → 向用户确认"迁移完成"。

---

## 五、对话模板（参考）

### 检测到需要迁移
> 当前数据 schema 版本为 1.0.0，skill 版本为 1.1.0。需要执行以下迁移：
>
> **v1.1.0 迁移内容**：
> 1. INDEX 添加 data_schema 字段，去掉"上次月度蒸馏"段
> 2. _align-log.md 内容转写到 journal 后删除
> 3. distill/ 目录删除
> 4. active.md / done-*.md 去掉顶部统计元数据
> 5. review 文件保持不动（兼容读取）
>
> 要执行吗？

### 迁移完成
> ✅ 数据已迁移到 v1.1.0。
> - _align-log.md 的 N 条记录已转写到 journal
> - distill/ 目录已删除
> - active.md / done-*.md 已简化
> - 旧 review 文件保留不动，新文件会用新格式
>
> 所有验证通过。

### 已是最新版本
> 当前数据 schema 已与 skill 版本一致（最新为 2.0.0），无需迁移。

---

### v1.2.0（从 v1.1.0 升级）

#### 新增

- INDEX.md 新增 `proactive: {nudge: on, strategy: on}` 字段
- INDEX.md 新增 `## Strategy Projects` 段（初始为空表格）
- INDEX.md 新增 `## Nudge 冷却` 段（初始为空）
- 新建 `$TASKOS_ROOT/profile.md`（空模板，completeness: 0）
- journal 新增 `[nudge]` 和 `[strategy]` 标记
- project 支持 `type: strategy`（不受 deadline 90 天约束，不占 core 槽位）

#### 一次性迁移动作

```
1. 在 INDEX.md 元数据区 `data_schema` 行后添加：
   proactive:
     nudge: on
     strategy: on

2. 在 INDEX.md "## Tasks Pool 概览" 段前添加：
   ## Strategy Projects
   （空）

3. 在 INDEX.md 末尾（"已完成本周"段之后）添加：
   ## Nudge 冷却
   （空）

4. 创建 `$TASKOS_ROOT/profile.md`，内容为空模板：
   ---
   type: user_profile
   created: <今天>
   last_updated: <今天>
   completeness: 0
   ---
   # 用户画像
   （各段落为空，等待"认识你"对话填充）

5. 更新 INDEX.md: `data_schema: 1.2.0`
```

#### 行为变更（迁移后自动生效）

- 启动行为新增 4 步：profile 读取 + nudge 扫描 + strategy 检视提醒 + profile 冷启动检测
- 启动询问优先级规则生效（避免同时轰炸用户）
- 周计划新增 nudge 段 + strategy 关联检查
- 周复盘新增 nudge 段 + strategy 关联检查
- 复盘末尾可触发 strategy 检视提醒
- 全面核查从 10 项增至 13 项
- 强制规则新增第 6 条（AI 建议不得自动写入）

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| INDEX.data_schema | == "1.2.0" |
| INDEX 含 proactive 字段 | nudge: on, strategy: on |
| INDEX 含 Strategy Projects 段 | 存在（可为空） |
| INDEX 含 Nudge 冷却段 | 存在（可为空） |
| profile.md | 文件存在，frontmatter 含 type: user_profile |

全部通过 → 向用户确认"迁移完成"。

---

### v1.2.2（从 v1.2.0/v1.2.1 升级）

#### 新增

- project frontmatter 支持 `justification` 字段（可选，用于记录项目准入理由）
- journal 新增 `[gatekeeper]` 和 `[gatekeeper-override]` 标记
- 新增 Gatekeeper 质询流程（创建 project 前强制触发）
- 全面核查新增第 14 项：project justification 完整性
- 强制规则新增第 7 条（Gatekeeper 机制）

#### 一次性迁移动作

```
1. 更新 INDEX.md: `data_schema: 1.2.2`

2. （可选）对现有 active project 执行存量审查：
   - AI 逐个用准入三条件+排除红线评估
   - 不符合标准的向用户质询理由
   - 通过的补充 justification 字段到 project frontmatter
   - 不充分的建议归档/删除
   - 审查结果走 journal [gatekeeper]
```

注：存量审查为可选。旧项目即使没有 justification 字段也不影响系统运行。全面核查第 14 项对存量审查前的旧项目豁免。

#### 行为变更（迁移后自动生效）

- 新建 project 前强制触发 Gatekeeper 质询流程
- AI 有权拒绝理由不充分的项目创建请求
- 全面核查从 13 项增至 14 项

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| INDEX.data_schema | == "1.2.2" |

全部通过 → 向用户确认"迁移完成"。

---

### v1.2.3（从 v1.2.2 升级）

#### 新增

- 人设段重写（核心信念增强：韧性 > 效率、留白空间、不上纲上线、不帮找借口）
- 周计划流程新增 §1.2b 韧性检测（连续高强度预警）
- 周计划流程新增 §1.8b 探索空间提醒
- Strategy 检视流程新增 §4b 模式性阈值（区分波动与模式）
- journal 新增 `[nudge-韧性-拒绝]` 和 `[nudge-留白]` 标记

#### 一次性迁移动作

无。本版本为纯行为优化，无数据结构变更。

```
1. 更新 INDEX.md: `data_schema: 1.2.3`（可选，非必须）
```

注：由于无 schema 变更，data_schema 升级为可选。不升也不影响功能。

#### 行为变更（升级 skill 文件后自动生效）

- AI 人设基调变化：从"在乎效率也在乎人"转为"韧性优先、允许留白、不上纲上线但也不帮找借口"
- 周计划：精力状态询问后增加韧性检测（连续高强度时主动建议降档）
- 周计划：排期完成后增加探索空间提醒（连续满排时提醒 TaskOS 之外还有生活）
- Strategy 检视：偏离度 10~20% 时区分单次波动和模式性偏离，避免过度反应

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.2.3" |
| SKILL.md「你是谁」段 | 含"韧性优先于效率"信念 |
| workflow-weekly.md | 含 §1.2b 韧性检测 + §1.8b 探索空间提醒 |
| workflow-strategy.md | 含 §4b 模式性阈值 |

全部通过 → 向用户确认"迁移完成"。

---

### v1.2.4（从 v1.2.3 升级）

#### 新增

- task JSONL 新增可选字段 `status`（not_started/in_progress/blocked）和 `note`（自由文本）
- 强制规则新增第 8 条（用户报告进度时必须即时写回 active.md）
- 周复盘快照新增 `[~]` 和 `[!]` 标记（部分完成 / 阻塞）
- Tasks Pool 概览新增"本周排期"计数行

#### 移除

- INDEX.md 当周快照段（## 当前周 ... ### 必须做 / 该做 / 可以做 / 已完成本周）
- workflow-healthcheck.md 核查清单原第 4 项（14 项变 13 项）

#### 一次性迁移动作

```
1. 删除 INDEX.md 中从 "## 当前周" 到 "## Nudge 冷却" 之前的所有内容
2. 在 Tasks Pool 概览段"active 总数"行后添加：
   - 本周排期: N 条（must: N, should: N, could: N）
3. 更新 INDEX.md: data_schema: 1.2.4
```

#### 行为变更（升级 skill 文件后自动生效）

- 所有任务查询直接从 active.md / done-*.md 实时计算
- 写操作改为刷新 Tasks Pool 概览计数（不再重新生成快照段）
- 用户报告进度时 AI 必须即时更新 active.md 的 status/note
- 周复盘快照区分 完成[x] / 部分完成[~] / 未开始[ ] / 阻塞[!]
- Nudge 扫描条件 d 排除 blocked 任务

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| INDEX.md 无"## 当前周"段 | 确认已删除 |
| INDEX.md Tasks Pool 概览有"本周排期"行 | 确认存在 |
| INDEX.data_schema | == "1.2.4" |
| SKILL.md version | == "1.2.4" |

全部通过 → 向用户确认"迁移完成"。

---

### v1.2.5（从 v1.2.4 升级）

#### 新增

- INDEX.md 新增 `weekly_est_limit_source: auto` 字段（auto | manual）
- INDEX.md 新增 `work_hours_this_week: null` 字段（本周全职工时，每次开周计划时更新）
- 周快照 `reviews/YYYY-Www.md` frontmatter 新增 `work_hours` 字段
- profile.md 新增「工作量基线」段（AI 自动维护，含外部参考 + 全职数据 + 个人历史指标）
- 周计划流程新增 §1.2c（本周全职工时采集）
- 周计划 §1.8 重写为双锚点校准逻辑（研究基准 + 个人历史）
- 强制规则新增第 9 条（用户报告全职变化时必须即时更新 profile）
- journal 新增 `[profile-workload-update]` 标记

#### 一次性迁移动作

```
1. 在 INDEX.md 元数据区 `weekly_est_limit` 行后添加：
   weekly_est_limit_source: auto
   work_hours_this_week: null

2. 在 profile.md 正文末尾添加「工作量基线」段（空模板）：
   ## 工作量基线（AI 自动维护，每月更新一次）
   ### 外部参考
   - 深度工作日上限: 4h/天，20~28h/周（Ericsson 1993）
   - 总认知工时最佳区间: 25~30h/周（Melbourne 2016）
   - 全职后业余深度参考区间: _~_h/周
   ### 全职工作数据
   - 典型周工作时长: _h
   - 工作认知强度: _
   - 典型通勤时长: _h/天
   - 本段最近更新: <今天>
   ### 个人历史指标（AI 从快照自动计算）
   - 数据窗口: 近 0 周（待积累）
   - 平均完成率: --%
   - 甜点排期量: --h/周
   - 各精力档完成率: high --% / normal --% / low --%
   - 过载周占比: --%
   - 平均全职工时: --h/周
   - 长期趋势: --
   - 本段最近更新: <今天>

3. 更新 INDEX.md: data_schema: 1.2.5
```

注：个人历史指标初始为空。积累 4 周快照后 AI 首次自动计算并填充。

#### 行为变更（升级 skill 文件后自动生效）

- 每次开周计划时 AI 主动询问本周全职工时（§1.2c），写入 INDEX
- 周计划 §1.8 从单阈值对比升级为双锚点校准（研究基准 + 个人历史甜点）
- AI 每月自动更新 profile 工作量基线（无需用户确认）
- 用户报告全职变化时 AI 必须即时更新 profile（强制规则 #9）
- 排期建议引用具体数据（"你的历史甜点是 Xh""研究参考上限是 Yh"）
- 全职 > 52h 时主动建议大幅减排

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| INDEX.md 含 weekly_est_limit_source | == "auto" |
| INDEX.md 含 work_hours_this_week | 字段存在（值可为 null） |
| profile.md 含「工作量基线」段 | 确认存在 |
| SKILL.md version | == "1.2.5" |
| INDEX.data_schema | == "1.2.5" |

全部通过 → 向用户确认"迁移完成"。

---

### v1.3.0（从 v1.2.5 升级）

#### 性质

架构重构（规则/指导分层），**无数据结构变更**。

#### 变更说明

- SKILL.md 新增「设计哲学」段，显式区分 ⚙️ 规则层和 💡 指导层
- workflow-weekly.md 从 ~420 行精确脚本重写为 ~200 行分层结构
- 原强制规则 #9（全职信息更新）降级为「设计哲学」段中的行为期望（非强制）
- 所有固定话术模板删除，替换为语气指导
- 阈值从硬编码改为"建议初始值 + 调整原则"

#### 一次性迁移动作

无。纯 skill 行为变更，无数据格式改动。

```
1.（可选）更新 INDEX.md: data_schema: 1.3.0
```

#### 行为变更

- AI 获得交互灵活性：可合并问题、调序、自选话术
- 信息采集不强制按步骤追问（用户已给的信息不重复问）
- 阈值可根据上下文偏移 10~20%
- 强制规则从 9 条精简为 8 条（保障数据一致性的底线不变）

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.3.0" |
| SKILL.md 含「设计哲学」段 | ⚙️ RULE / 💡 GUIDE 分层说明存在 |
| workflow-weekly.md | 含 ⚙️ 必须完成的动作 + 💡 指导层标注 |
| 强制规则条数 | 8 条 |

全部通过 → 向用户确认"迁移完成"。

---

### v1.4.0（从 v1.3.0 升级）

#### 性质

功能性新增（许愿卡奖励系统），**无现有数据结构变更**。

#### 新增

- 新增 `rewards/` 目录及 5 个文件（ledger/bounties/wishlist/streaks/challenges）
- INDEX.md 新增 `## Wish Cards` 段
- schema.md 新增第十三节（许愿卡数据格式）
- 新建 workflow-wishcard.md（许愿卡完整工作流）
- workflow-weekly.md 周复盘新增第 7 步（许愿卡周结算）
- workflow-weekly.md 周计划新增末尾步骤（提出本周挑战）
- SKILL.md 触发词新增许愿卡相关
- SKILL.md 工作流路由新增 workflow-wishcard.md
- journal 新增多个标记（wish-card-earn/spend、bounty-*、streak-*、challenge-*、wishlist-add）

#### 一次性迁移动作

**无强制迁移**。许愿卡系统在用户首次触发相关指令时自动初始化。

```
1.（可选）更新 INDEX.md: data_schema: 1.4.0
```

#### 行为变更（升级 skill 文件后自动生效）

- 用户说"许愿卡/奖励自己/打卡/悬赏/挑战"等触发词时进入许愿卡工作流
- 周计划完成后 AI 提出本周挑战（rewards/ 不存在则跳过）
- 周复盘末尾新增许愿卡周结算（rewards/ 不存在则跳过）
- 所有许愿卡操作遵循懒加载原则（不增加启动行为负担）

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.4.0" |
| SKILL.md 触发词含"许愿卡" | ✅ |
| SKILL.md 工作流路由含 workflow-wishcard.md | ✅ |
| workflow-wishcard.md | 文件存在 |
| workflow-weekly.md 周复盘含第 7 步 | ✅ |
| schema.md 含第十三节（许愿卡数据格式） | ✅ |
| migration.md 含 v1.4.0 段 | ✅ |

全部通过 → 向用户确认"迁移完成"。

---

### v1.5.0（从 v1.4.0 升级）

#### 性质

行为增强（心理增强层），**无数据结构强制变更**。新字段在首次周计划时自动初始化。

#### 新增

- INDEX.md 新增 `mood_this_week: null` 字段（本周情绪底色）
- INDEX.md 新增 `last_value_audit: null` 字段（上次价值审计日期）
- 周快照 frontmatter 新增 `mood` 字段（从 INDEX 取值）
- 周快照 frontmatter 新增 `recovery_week` 字段（恢复周标记）
- 周快照正文可能新增 `## 本周反思` 段和 `## 月度反思` 段
- 周计划新增恢复协议检测（连续高压/低迷时建议恢复周）
- 周计划信息采集新增情绪底色询问（可选）
- 周计划排期改为 AI 预排草案模式（用户审核确认）
- 周计划滞留处理改为决策合批模式（用户审核确认）
- 周复盘新增进展可视化（文本进度条对比）
- 周复盘新增微反思问题（每周 1 个，从问题池轮换）
- 每 4 周新增月度抽象化问题
- 复盘/回顾时可触发价值对齐审计（约每 12 周）
- progress-update 时 AI 附加进展对比叙事
- profile.md 工作量基线更新逻辑排除 recovery_week 快照

#### 一次性迁移动作

**无强制迁移**。新字段在首次周计划时自动初始化为 null/false。

```
1.（可选）在 INDEX.md 元数据区添加：
   mood_this_week: null
   last_value_audit: null

2.（可选）更新 INDEX.md: data_schema: 1.5.0
```

#### 行为变更（升级 skill 文件后自动生效）

- 周计划开头检测恢复协议条件（快照不足时跳过）
- 周计划信息采集时温和询问情绪底色（不强制）
- 周计划排期改为"AI 提供草案 → 用户审核确认"模式
- 周计划滞留处理改为批量建议模式
- 周复盘新增进展可视化展示（文本进度条）
- 周复盘末尾新增微反思问题
- 每 4 周新增月度抽象化
- progress-update 时附加进展对比
- 约每 12 周触发价值对齐审计
- 恢复周的快照标记 recovery_week: true，不计入甜点值

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.5.0" |
| workflow-weekly.md 含恢复协议段 | ✅ |
| workflow-weekly.md 含情绪采集 | ✅ |
| workflow-weekly.md 含 AI 预排草案 | ✅ |
| workflow-weekly.md 含决策合批 | ✅ |
| workflow-weekly.md 含进展可视化 | ✅ |
| workflow-weekly.md 含微反思 | ✅ |
| workflow-weekly.md 含月度抽象 | ✅ |
| workflow-weekly.md 含恢复周复盘调整 | ✅ |
| workflow-retrospect.md 含价值审计 | ✅ |
| schema.md INDEX 含 mood_this_week | ✅ |
| schema.md INDEX 含 last_value_audit | ✅ |
| schema.md 快照含 mood + recovery_week | ✅ |
| migration.md 含 v1.5.0 段 | ✅ |

全部通过 → 向用户确认"迁移完成"。

---

### v1.5.1（从 v1.5.0 升级）

#### 性质

行为增强（随手反思），**无数据结构强制变更**。新增独立文件 `reflections.md`，无 INDEX 字段变更。

#### 新增

- 新增 `reflections.md`（随手反思滚动文件，标题 + JSONL 区块）
- 新增触发词：记个反思 / 随手反思 / 想到一件事 / 记一下感想 / 突然想到
- 新增 `references/workflow-reflect.md` 工作流
- 周复盘微反思环节新增"先汇总本周散记"步骤
- ID 规范新增 `refl-YYYYMMDD-NNN` 格式

#### 一次性迁移动作

**无强制迁移**。`reflections.md` 在用户首次触发随手反思时自动创建（不存在则建）。

```
无需任何手工操作。data_schema 保持 1.5.0 不变
（随手反思为行为增强，新增独立文件，不涉及 INDEX/快照字段变更）。
```

#### 行为变更（升级 skill 文件后自动生效）

- 用户可随时用反思触发词记录想法到 reflections.md
- "记一下 XX"仍默认走 capture（向后兼容）；"记个反思/随手反思"等才走反思工作流
- 周复盘微反思前先回顾并汇总本周散记进快照 `## 本周反思` 段

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.5.1" |
| SKILL.md 触发场景含反思触发词 | ✅ |
| SKILL.md 工作流路由含 workflow-reflect.md | ✅ |
| SKILL.md 即时核查含随手反思行 | ✅ |
| SKILL.md 参考文件路由含 workflow-reflect.md | ✅ |
| workflow-reflect.md 存在且含 capture 区分规则 | ✅ |
| workflow-weekly.md 周复盘含"汇总本周散记"步骤 | ✅ |
| schema.md 文件清单含 reflections.md | ✅ |
| schema.md 含随手反思数据格式段（§十四） | ✅ |
| schema.md ID 规范含 refl- 格式 | ✅ |
| migration.md 含 v1.5.1 段 | ✅ |

全部通过 → 向用户确认"迁移完成"。

---

### v1.7.0（从 v1.5.1 升级，合并 v1.6.0 + v1.7.0）

#### 性质

合并升级：v1.6.0（三档排期时间制）+ v1.7.0（习惯打卡系统）。**新增可选字段 + 新增独立文件，无破坏性迁移**。data_schema 升 1.7.0。

#### 新增（v1.6.0 三档排期时间制）

- INDEX.md 新增 `weekly_tier_limits` 段（must: 15h、should: null、could: null）
- INDEX.md 新增 `tier_limits_source: default`
- 三档排期上限从"限条数"改为"限时间"：周总上限 weekly_est_limit（默认上调到 32h）+ must/should 时间子上限 + could 纯弹性
- 排期时对 must/should 缺 est 任务做推断或问询，兜底 1h
- 精力档从"查死表定条数"改为指导层浮动比例

#### 新增（v1.7.0 习惯打卡系统）

- 新增 `habits.md`（习惯打卡，标题 + JSONL，懒加载）
- INDEX.md 新增 `## Habits 概览` 段
- 新增触发词：打卡 / 习惯打卡 / 加个习惯 / 我的习惯 / 习惯毕业 / 升到核心层 / 降到观察层
- 新增 `references/workflow-habit.md` 工作流
- ID 规范新增 `h-YYYYMMDD-NNN`
- journal 新增 `[habit-create]` / `[habit-checkin]` / `[habit-graduate]` 标记
- 许愿卡 earn source 新增 `habit-graduate`（核心习惯毕业发 1 张）

#### 一次性迁移动作

```
1. INDEX.md 元数据区在 weekly_est_limit 行后添加：
   weekly_est_limit_source: manual    （若已有则保留）
   weekly_tier_limits:
     must: 15h
     should: null
     could: null
   tier_limits_source: default
2. （可选）将 weekly_est_limit 从旧默认 12h 上调到 32h，或在首次开周计划时由 AI 引导确认
3. INDEX.md 末尾添加 `## Habits 概览` 段（核心层: 0/3 / 观察层: 0），或首次用到时自动建
4. 更新 INDEX.md: data_schema: 1.7.0
```

> `habits.md` 在用户首次创建习惯/打卡时自动创建（不存在则建），无需预建。

#### 行为变更（升级 skill 文件后自动生效）

- 周计划三档排期改为时间制；首次排期 AI 引导确认三档时间额度（must 15h / should 8h 建议 / could 不限 / 总 32h）
- 精力低的周 AI 按比例建议下调时间额度
- 新增习惯打卡：核心层（硬上限 3 个，每日批量打卡"默认全过只报例外"）+ 观察层（不限量，只累计不断签）
- 核心习惯毕业（~66 天锚点，可拒绝）→ 旋转门腾位 + 发 1 张许愿卡
- `core 项目 ≤ 3` 约束保持不变（与三档排期无关）

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.7.0" |
| INDEX 含 weekly_tier_limits + tier_limits_source | ✅ |
| weekly_tier_limits.could 恒为 null | ✅ |
| INDEX 含 ## Habits 概览 段 | ✅ |
| SKILL.md 触发场景 + 路由含习惯打卡 / workflow-habit.md | ✅ |
| workflow-habit.md 存在且含批量打卡 + 与 Streak 划界 | ✅ |
| workflow-weekly.md 三档表为时间制 + 含 est 缺失处理 | ✅ |
| schema.md 含习惯数据格式（§十五）+ h- ID + weekly_tier_limits | ✅ |
| schema.md journal 标记含 habit-create/checkin/graduate | ✅ |
| workflow-wishcard.md earn source 含 habit-graduate | ✅ |
| `core 项目 ≤ 3` 约束仍存在且未改 | ✅ |
| INDEX.data_schema | == "1.7.0" |

全部通过 → 向用户确认"迁移完成"。

---

### v1.7.1（从 v1.7.0 升级）

#### 性质

纯文档维护 + 行为规则补充，**无数据格式变更，无一次性迁移动作**。data_schema 保持 1.7.0。升级 skill 文件即可，老数据零改动。

#### 变更内容

1. **默认中文对话规则**：SKILL.md 强制规则新增第 9 条——默认用简体中文与用户对话，除非用户当前对话明确要求换语言。仅约束对话语言，不影响数据文件中必须保留的英文（字段名/枚举/ID/文件名/journal 标记）。
2. **中英文梳理**：把文档中可中文化的英文叙述统一中文化——
   - 各 references 章节标题 `# Workflow: Xxx` → `# 工作流：Xxx`
   - `Mini-Check` → 即时核查（全 skill 统一）
   - `Recovery Protocol` → 恢复协议；散文中 `Weekly Plan/Review` 引用 → 周计划/周复盘
   - **保留英文不变**：数据锚点（字段名、枚举值、ID 格式、文件名、`JSONL/YAML/INDEX`、journal 标记、INDEX 段标题如 `## Nudge 冷却`/`## Strategy Projects`、数据文件标题如 `# Active Tasks`/`# Reflections`/`# Habits`/`# Wish Cards`）
3. **healthcheck.md 内部一致性修复**：项数全文统一为 13 项；删除历史遗留的"INDEX 当周快照一致性"残留检查（v1.2.4 已移除该段）；步骤/范例/模板同步校正。

#### 行为变更（升级 skill 文件后自动生效）

- AI 默认全程用简体中文与用户交互
- 文档术语统一，无功能行为变化

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.7.1" |
| SKILL.md 强制规则含第 9 条「默认简体中文对话」 | ✅ |
| 全 skill 无 `Workflow:` / `Mini-Check` / `Recovery Protocol` 英文残留 | ✅ |
| 数据锚点（字段/枚举/ID/文件名/journal 标记/INDEX 段标题）未被误改 | ✅ |
| workflow-healthcheck.md 项数全文一致为 13 项 | ✅ |
| INDEX.data_schema | == "1.7.0"（不变） |

全部通过 → 向用户确认"迁移完成"。

---

### v1.8.0（从 v1.7.1 升级）

#### 性质

机制收敛 + 减负 + bug 修正。涉及 INDEX 字段删除（`active_streaks`）与可选文件退役（`rewards/streaks.md`），故 data_schema 升 1.8.0。**不破坏 active 中央池；迁移轻量幂等。**

#### 变更内容

1. **许愿卡 Streak 退役**：删除连续纪录子系统（streaks.md + 工作流）。连续打卡能力统一由习惯系统（habits.md）承接；"连续达成攒卡"由悬赏（bounty）承接；"打卡"触发词统一路由习惯系统。
   - earn `source` 枚举删除 `streak`（保留 `direct/bounty/challenge/habit-graduate`）
   - Bounties `auto_detect` 枚举删除 `"streak:s-ID"`
   - ID 规范删除 `s-YYYYMMDD-NNN`
   - INDEX Wish Cards 段删除 `active_streaks`
   - journal 标记删除 `streak-create/streak-checkin/streak-restart`
2. **启动行为压缩**：11 步 → 3 步必做硬地板（读 INDEX / 崩溃恢复 / current_week 比对）+ 7 项按需触发（无信号静默跳过）。能力零损失。
3. **bug 修正**：
   - 启动计数校验公式加 Strategy 段（原 Core+Normal+Side 三段，导致有 strategy 项目时每次误报漂移）
   - SKILL.md 参考路由"14 项"→"13 项"（与 healthcheck 一致）
   - `weekly_est_limit_source` 示例统一为 manual

#### 一次性迁移动作（手动触发，幂等）

```
1. rewards/streaks.md 处理：
   - 不存在 → 无操作（绝大多数情况）
   - 存在但 JSONL 区块为空 → 删除该文件，journal [align]
   - 存在且有 active 连续纪录 → 询问用户：
     "连续纪录功能已退役并入习惯系统。你有 N 条连续纪录，
      建议转成习惯核心层（保留连续天数），或设成悬赏。要怎么处理？"
     · 转习惯 → habits.md 建对应核心层习惯（streak←current_count，
       best_streak←best_count，total 估为 current_count），journal [habit-create]
     · 设悬赏 / 不要了 → 归档 streaks.md 到 rewards/archive/（不删数据）
2. INDEX Wish Cards 段删除 active_streaks 行（若存在）
3. data_schema → 1.8.0
```

> 多数用户从未初始化 streaks.md，步骤 1 通常无操作。归档不删数据。

#### 行为变更（升级 skill 文件后自动生效）

- "打卡"统一进习惯系统，不再触发许愿卡 Streak
- 启动从 11 步无条件 → 3 步必做 + 7 项按需触发
- 启动计数校验含 Strategy 段，不再误报漂移
- 周复盘许愿卡结算不含 streak（本就只读 bounty/challenge/ledger）

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "1.8.0" |
| SKILL.md 启动行为为"3 必做 + 7 按需"结构 | ✅ |
| SKILL.md 启动计数校验含 Strategy 段（四段） | ✅ |
| SKILL.md 参考路由"全面核查"为 13 项 | ✅ |
| SKILL.md 触发场景/路由无 streak 专属词、"打卡"仅归习惯 | ✅ |
| workflow-wishcard.md 无"连续纪录（Streak）"段、earn source 无 streak | ✅ |
| workflow-habit.md 划界表为 2 行、无"与 Streak 消歧"段 | ✅ |
| schema.md 文件清单无 streaks.md、无 Streaks 行格式表、无 s- ID、无 active_streaks | ✅ |
| schema.md Bounties auto_detect 无 "streak:s-ID" | ✅ |
| schema.md 启动校验含 Strategy 四段、weekly_est_limit_source 示例 == manual | ✅ |
| 习惯系统/ritual/强制规则条数（9 条）未被误改 | ✅ |
| migration.md 含 v1.8.0 段 | ✅ |
| INDEX.data_schema | == "1.8.0" |

全部通过 → 向用户确认"迁移完成"。

---

### v2.0.0（从 v1.8.x 升级）

#### 性质

大规模功能删减：聚焦长程项目管理。删除许愿卡奖励系统、习惯打卡系统、Gatekeeper 门禁、随手反思、反思增强 5 个子系统。

#### 删除

- 子系统：许愿卡奖励系统（rewards/）、习惯打卡（habits.md）、Gatekeeper 门禁、随手反思（reflections.md）、反思增强（心理增强层子机制）
- 文件：`references/workflow-wishcard.md`、`references/workflow-habit.md`、`references/workflow-reflect.md`
- INDEX 字段：Wish Cards 段、Habits 概览段
- journal 标记：gatekeeper / gatekeeper-override / reflect / habit-* / wish-card-* / bounty-* / challenge-* / wishlist-add
- ID 格式：wc- / b- / ch- / refl- / h-
- 强制规则：9 条→8 条（删除原 #7 Gatekeeper，后续重新编号）

#### 一次性迁移动作（手动触发，幂等）

```
1. rewards/ 目录 → 若存在则询问是否归档到 rewards_archive/
2. habits.md → 若存在则询问是否归档
3. reflections.md → 若存在则询问是否归档
4. INDEX.md 清理 Wish Cards 段和 Habits 概览段
5. INDEX.data_schema → 2.0.0
6. .journal.md：历史记录不动
```

> 废弃文件均归档不删除。

#### 验证清单

| 检查项 | 预期 |
|--------|------|
| SKILL.md version | == "2.0.0" |
| SKILL.md 无 Gatekeeper 段、强制规则 8 条 | ✅ |
| SKILL.md 工作流路由无 reflect/habit/wishcard | ✅ |
| references/ 下无 workflow-wishcard/habit/reflect.md | ✅ |
| schema.md data_schema == "2.0.0" | ✅ |
| schema.md 无 rewards/habits/reflections 数据格式 | ✅ |
| workflow-weekly.md 无许愿卡结算/微反思 | ✅ |
| workflow-healthcheck.md 13 项精简版 | ✅ |
| migration.md 含 v2.0.0 段 | ✅ |
| INDEX.data_schema | == "2.0.0" |

全部通过 → 向用户确认"迁移完成"。
