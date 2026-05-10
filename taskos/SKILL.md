---
name: taskos
description: "通用个人任务管理 skill：基于 Areas/Projects/Tasks 三层 SOP + 优先级 + 风险驱动 + 懒人友好 + JSONL 中央池的目标推进系统。任务永不丢失，跨 AI agent 可移植。"
description_zh: "通用个人任务管理 skill：基于「领域/项目/任务」三层 SOP，含核心项目优先级、风险驱动评估、懒人友好模式、JSONL 中央任务池。任务永续保留，跨 AI agent 可移植。"
description_en: "Universal personal task management skill: 3-tier SOP (Areas/Projects/Tasks) + priority + risk-driven assessment + lazy-mode friendly + JSONL central pool. Tasks never lost, portable across AI agents."
version: 0.9.4
license: MIT
metadata:
  category: productivity
  agent_created: true
---

# TaskOS — 通用个人任务管理 Skill

本 skill 基于「领域 (Areas) → 项目 (Projects) → 任务 (Tasks)」三层 SOP，帮助用户管理长期目标和本周行动。**任务永续保留在 JSONL 中央池，不会随时间消失**。

---

# 数据根目录（换设备时改这一行）

```
TASKOS_ROOT: C:\Users\jqdzhang\TaskOS
```

> 不同 OS 下示例：
> - Windows: `C:\Users\jqdzhang\TaskOS`
> - macOS / Linux: `~/TaskOS` 或 `/Users/jqdzhang/TaskOS`
>
> 路径变更只需改这一行；其他文件中所有路径用相对路径。

---

# ⚠️ 不可违反的强制规则

```
1. 每次响应用户问题前，必须先读 INDEX.md。
   不允许基于"上次对话里的记忆"或 token 缓存回答。
   即便用户连续问 10 个相关问题，也要每次都重新读 INDEX。

2. 任何写操作（即便只动 1 个文件）都必须 append .journal.md 的 [in_progress] 行；
   操作完成后必须 append [done] 行。

3. 每次写操作后必须刷新 INDEX 的 last_updated、version，
   并重算被改动的 active.md / done-*.md 顶部元数据（last_updated/total/by_project_count）。

4. 启动时如检测到 .journal.md 末尾有未配对 [in_progress]，
   必须主动告知用户并询问处理方式（A 继续 / B 回滚 / C 标记已完成）。

5. 启动时比对 INDEX.current_week 与今天的 ISO 周（用日期计算，不能字符串递增）；
   不一致时主动询问用户"要不要现在开周计划？"；
   若用户拒绝，AI 自动更新 INDEX.current_week 为今天 ISO 周并重生成当前周段（走 journal）。
```

---

# 触发场景

用户提到以下任一时，进入对应工作流：
- 任务、项目、本周计划、周复盘、领域规划、改名
- "记一下…" / "加任务" / "我现在要做什么" / "还有什么没做" / "把 X 归到 Y"

---

# 启动行为（每次调用 skill 都跑）

按顺序执行：

1. 检查 `${TASKOS_ROOT}` 是否存在
   - 不存在 → 询问用户是否初始化
   - 用户同意 → 创建空骨架（详见末尾"初始化"段）

2. 全量读 `INDEX.md` 加载到上下文

3. **轻量计数校验**（替代 fingerprint）：
   - active.md JSONL 行数 vs INDEX `active 总数`
   - `areas/` 目录下 .md 文件数 vs INDEX 列出的 area 数
   - `projects/active/` 目录下 .md 文件数 vs INDEX 中 Core + Normal + Side 三段的项目数总和
   - 任一不一致 → 提示用户"INDEX 可能漂移，建议立即重建"，并询问是否当场重建

4. 检查 `.journal.md` 末尾是否有未配对 `[in_progress]`
   - 有 → 告知用户处理方式（详见强制规则 #4）

5. 比对 `INDEX.current_week` 与今天的 ISO 周（用日期计算）
   - 不一致 → 主动询问"要不要现在开周计划？"
   - 用户同意 → 进入 weekly plan 工作流
   - 用户拒绝 → AI 自动只更新 INDEX.current_week 为今天 ISO 周，重生成当前周段（基于 active.md 中 due_week == 新 current_week 的任务，走 journal）

6. 检查月度蒸馏是否过期（INDEX 中"上次月度蒸馏"早于本月）
   - 过期 → 自动跑 distill 工作流

7. 检查 `.journal.md` 是否需要月度归档
   - 最早记录早于本月 → 切到 `.journal-YYYY-MM.md`，当月留在 `.journal.md`

8. 懒人模式检测
   - 计算今天 - INDEX.last_weekly_plan
   - > 7 天 → 标记下次 weekly plan 走对齐流程（详见 workflow-weekly.md）

---

# 工作流路由

| 触发词 | 进入工作流 |
|---|---|
| 记一下 / 加任务 / 帮我归类 / 把 X 归到 Y | references/workflow-capture.md |
| 开周计划 / 这周要做什么 / 排一下本周 | references/workflow-weekly.md（plan 部分） |
| 周复盘 / 本周怎么样 / 复盘一下 | references/workflow-weekly.md（review 部分） |
| 把 X 改名为 Y / 重命名 X | references/workflow-rename.md |
| （月度蒸馏过期） | references/workflow-distill.md（启动行为自动触发） |

---

# 高频查询专用入口（只读路径）

触发：我现在要做什么 / 现在做啥 / 还有什么没做 / X 项目进展怎样 / 这周还剩什么 / 拖了多久没做的

**SOP（极简，只读不写）**：

1. 强制读 INDEX.md（执行强制规则 #1）
2. 答案路径：
   - "本周还有什么没做" → 读 INDEX 当前周快照段（已聚合）
   - "拖了多久没做的" → 读 active.md，列出 `(carry ≥ 1) ∪ (滞留任务)` 按 ID 去重
   - "X 项目进展" → 读对应 project 文件 + active.md 中 projects 数组含 X 的任务 + 当月及上月 done-YYYY-MM.md 中含 X 的任务（完整呈现"已做 N 条 + 未做 M 条"）
   - "我现在要做什么" → INDEX 当前周快照"必须做"档
3. **不写任何文件**，不更新 journal，不更新 INDEX
4. 直接回答

**好处**：高频查询不污染 .journal、不消耗写入开销、token 成本极低、即便用户连问 20 次也不会出问题。

---

# Mini-Check 表（每次写操作后立即跑）

| 本次操作 | 检查 |
|---|---|
| Capture 一条 task 到 active.md | id 全局唯一（active + 当 captured 月 done + inbox）；引用的 project 是否存在；动词前置改写完成；active.md 顶部 total/by_project_count 重算；INDEX 刷新 |
| Capture 一条 task 到 inbox.md | id 全局唯一；动词前置改写完成；captured 日期已填；INDEX 刷新 |
| Inbox 整理（移到 active） | inbox.md 该行已删；active.md 已 append 且 ID 保留；缺失字段已补全（projects/area/carry=0/due_week=null/tier=null）；INDEX 刷新 |
| 完成一条 task | active.md 该行已删；done-YYYY-MM.md 已 append 全字段（含 null）+ completed + outcome:done + week；active.md 顶部元数据重算；INDEX 当前周"已完成本周"同步；INDEX version+1 |
| 放弃一条 task | active.md 该行已删；done-YYYY-MM.md 已 append 全字段 + completed（今天）+ outcome:dropped；INDEX 同步 |
| 修改 done 任务的 completed 日期 | 若新旧日期跨月 → 跨文件迁移（旧 done-*.md 删行，新 done-*.md append）；若 week 字段也变 → INDEX 当前周"已完成本周"重算；INDEX 同步 |
| 修改 done 任务的 outcome 字段 | done-*.md 整行替换；若涉及 INDEX 当前周显示 → 重生成；INDEX 同步 |
| 改 task 的 due_week / tier | active.md 整行替换；INDEX 当前周快照重生成 |
| 改 project status | INDEX 更新；archive 触发条件检查 |
| 改 priority 为 core | core 总数是否超 3（超出立即告警） |
| 改 progress | risk 是否需重算 |
| Rename | 全局检查旧名是否还有残留（除 done-*.md 外）；INDEX 同步；_align-log 是否记一笔；priority/status/area 等其他字段未被误改 |
| current_week 修正（启动行为自动） | 用户拒绝开 plan 时由 AI 自动修正；走 journal；当前周快照重生成 |

不一致 → 立即提示用户 + 给修复建议（不擅自修复）。

---

# 自动归档规则

由启动行为或写操作自动触发，静默执行：

- Project status = `done` 或 `dropped` 满 14 天 → 移入 `projects/archive/`
- Review 文件创建满 12 周 → 移入 `reviews/archive/`
- done-*.md **无切片机制**：任务完成时**直接根据 completed 日期写到对应月份文件**（不存在时自动创建）

---

# 优先级约束

- 同时 active 的 core 项目 ≤ 3 个
- 超出 → Mini-Check 立即告警（不阻断，但提示降级）

---

# 任务文件区分

| 文件 | 角色 |
|---|---|
| `tasks/inbox.md` | 未分类捕获区（脑暴草稿，markdown checklist 格式） |
| `tasks/active.md` | 已分类未完成的中央池，**永续**（JSONL 格式） |
| `tasks/done-YYYY-MM.md` | 已完成 / 放弃归档（JSONL，按 completed 月份直接写入） |

---

# 风险模型简表

```
gap = actual_progress - expected_progress

actual_progress  = project.progress 字段（用户主观给）
expected_progress = (今天 - created) / (deadline - created)

🟢 健康：gap ≥ -0.10
🟡 关注：-0.30 ≤ gap < -0.10
🔴 高危：gap < -0.30，或 (剩余天数 < 7 且 actual_progress < 0.7)

key_milestones（可选）触发：
- 剩余天数 < 50% 但全 pending → 强制 🔴
- 剩余天数 < 25% 但完成 < 50% → 强制 🔴

优先级加权：
- core 🟡 → 视为 🔴 提示
- side 🔴 → 仅 weekly review 时提示
```

---

# 参考文件路由

详细规则在 `references/` 下：

- `references/schema.md` — 完整数据模型 + JSONL 操作规则 + INDEX 格式 + ID 规范
- `references/workflow-capture.md` — 捕获分层 + inbox 整理详细 SOP
- `references/workflow-weekly.md` — 周计划 + 周复盘 + 风险模型 + 完整性扫描
- `references/workflow-rename.md` — 重命名工作流 + 旧名历史保留
- `references/workflow-distill.md` — 月度自动蒸馏（含 area 活跃度 / yearly_intent 进度）

模板在 `templates/`：
- `templates/area.md` — area 文件模板
- `templates/project.md` — project 文件模板
- `templates/active-jsonl-line.md` — JSONL 行格式范例 + 字段说明

遇到具体操作时按需 load 对应 reference。

---

# 跨 agent 兼容性

本 skill 不依赖任何特定 AI agent 的私有工具。仅依赖：
- 读文件 / 写文件 / 列目录
- Markdown / YAML / JSON 解析

可在任何兼容 Anthropic 风格 skill 的 agent（Claude Code、Cursor、OpenClaw、WorkBuddy 等）上加载使用。

> ⚠️ 实际部署到 OpenClaw 等其他 agent 时，需先验证：
> - skill 加载机制
> - 路径变量解析
> - 文件读写工具是否可用

---

# 初始化（首次使用 skill 时）

当用户第一次调用 skill 且 `${TASKOS_ROOT}` 不存在时，AI 主动询问：

> 检测到 TaskOS 数据目录还不存在（${TASKOS_ROOT}）。要不要现在初始化？
>
> 这会创建：
> - 目录结构：areas/、projects/active/、projects/archive/、tasks/、reviews/、reviews/distill/
> - 空文件：INDEX.md、tasks/inbox.md、tasks/active.md、.journal.md、reviews/_align-log.md、README.md
> - **不预填任何 area / project**——你可以之后用 capture 工作流逐步建立
>
> 初始化吗？

用户同意 → 按以下结构创建：

```
${TASKOS_ROOT}/
├── README.md
├── INDEX.md                 # 初始版本：current_week 设为今天 ISO 周，version: 1
├── .journal.md              # 仅一条初始化 [done] 记录
├── areas/                   # 空目录
├── projects/
│   ├── active/              # 空目录
│   └── archive/             # 空目录
├── tasks/
│   ├── inbox.md             # 仅含标题"# Inbox"
│   └── active.md            # 含完整 markdown 头 + 空 JSONL 区块
└── reviews/
    ├── _align-log.md        # 仅含标题"# Align Log"
    └── distill/             # 空目录
```

初始化的 INDEX.md 模板：
```markdown
# TaskOS Index
last_updated: <今天>
last_full_rebuild: <今天>
last_weekly_plan: null
version: 1
current_week: <今天的 ISO 周>

## Areas (active: 0)
（空）

## Projects — Core (0/3)
（空）

## Projects — Normal (0)
（空）

## Projects — Side (0)
（空）

## Tasks Pool 概览
- inbox: 0 条
- active 总数: 0
- 高 carry 任务（≥3）: 0 条
- 滞留任务: 0 条
- by_project: 空

## 当前周 <ISO 周>（生成于 <今天>）
range: <周一> ~ <周日>

### 必须做（must, ≤3）

### 该做（should, ≤7）

### 可以做（could, ≤5）

### 已完成本周

## 上次月度蒸馏
- last_distill: null    # 首次启动后会自动跑上月的 distill 工作流
```

初始化 active.md 模板：
```markdown
# Active Tasks
last_updated: <今天>
total: 0
by_project_count: {}

## ~~~ JSONL 区块开始 ~~~
## ~~~ JSONL 区块结束 ~~~
```

完成后 AI 提示：
> ✅ TaskOS 已初始化在 ${TASKOS_ROOT}。
>
> 接下来建议：
> 1. 创建第一个 area（直接说"加一个领域：心理学"）
> 2. 在该 area 下建第一个 project（说"在心理学下加项目：精读《XX》"）
> 3. capture 第一条 task（说"记一下：精读第 1 章"）

---

# 版本与维护

- 当前版本：v0.9.4（实施后审核修订：ID 唯一性扫描描述 / 计数校验措辞 / 风险模型字段名统一 / by_project_count 特殊 key 判定规则 / 初始化 distill 字段格式）
- 设计原则：精简稳定 + 高频可信 + 任务永续 + 跨 agent 可移植
- 已通过 5 轮严格审核 + 1 次实施后审核，零 P0 / P1 漏洞

如果在使用过程中发现实战问题，请：
1. 在 `_align-log.md` 留一笔反馈
2. 修订 reference 文件并升 minor 版本号（v0.9.4 → v0.9.5）
