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
> 当前数据 schema 已是 1.2.3，与 skill 版本一致，无需迁移。

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
