# 咚咚 🪔

> 一个会主动判断、会照顾你状态的 AI 任务管理伙伴。

## 目录

- `taskos/` — 咚咚 · 个人任务管理 Skill

```
taskos/                         咚咚 · 个人任务管理 Skill
├── SKILL.md                    入口：人设 / 强制规则 / 启动行为 / 工作流路由
├── references/                 详细工作流与数据规则（按需加载）
│   ├── schema.md               数据模型 + JSONL 操作 + INDEX 格式 + ID 规范
│   ├── migration.md            版本迁移指引
│   ├── workflow-capture.md     任务捕获 + inbox 整理
│   ├── workflow-weekly.md      周计划 + 周复盘 + 风险模型
│   ├── workflow-rename.md      重命名 + 旧名历史保留
│   ├── workflow-retrospect.md  手动复盘 + 价值对齐审计
│   ├── workflow-strategy.md    长期路线图（创建/研究/检视/调整）
│   ├── workflow-healthcheck.md 全面核查（一键体检）
│   └── workflow-cleanup.md     数据瘦身
└── templates/                  文件模板
    ├── area.md                 领域模板
    ├── project.md              项目模板
    └── active-jsonl-line.md    任务 JSONL 行格式范例
```

## 咚咚 Skill

基于「领域/项目/任务」三层 SOP 的通用个人任务管理 skill。

- 版本：v2.0.0
- 兼容：Claude Code / Cursor / OpenClaw / WorkBuddy 等支持 Anthropic 风格 skill 的 agent
- 数据格式：纯 Markdown + JSONL，跨 AI agent 可移植

## 设计理念

传统任务管理工具（滴答清单、Notion、Todoist）擅长记录、整理和提醒。TaskOS 在这个基础上加了一层：一个具有主动判断力、会照顾你状态的 AI 伙伴。

**传统工具做不到的几件事：**

- **会主动开口**：连续几周没推进重点项目，它直接说出来；连续高强度运转，它拦你一把。
- **温和重启**：长期没更新，回来时帮你平静地重新拾起，不靠愧疚驱动。
- **照顾状态**：采集情绪底色、设恢复协议、留韧性阈值，在意你扛得住扛不住、恢复得了恢复不了。
- **以周为颗粒度**：日粒度太细，一天没完成就产生挫败；月粒度太粗，反馈周期长到容易迷失。周是那个"既能看到进展、又允许某天状态不好"的甜蜜点——刚好贴合人自然的精力节律，也刚好是 GTD 体系中"周回顾"被验证最关键的节奏。
- **对话即操作**：自然语言交代，AI 维护三层结构，省去自己搭系统、调视图的功夫。
- **数据握在手里**：纯 Markdown + JSONL，任何支持 skill 的 agent 都能接手。

**方法论基础：**

- **PARA（Projects, Areas, Resources, Archives）**：TaskOS 采用其核心分层思想——用「领域 → 项目 → 任务」三层结构管理信息。领域承载长期身份和持续关注点（如"健康""职业发展"），项目承载有明确终点的目标，任务承载本周可执行的具体动作。三层各司其职，既看得见远方，也踩得稳脚下。

- **OKR（Objectives & Key Results）**：每个领域设有年度意图（yearly_intent），类似 OKR 中的 Objective——一个有方向感的愿景声明。它让日常的项目和任务始终能向上追溯到"我为什么在做这件事"，避免忙碌却迷失方向。

- **GTD（Getting Things Done）**：遵循 GTD 的核心原则——把所有想法快速捕捉到一个可信赖的系统中，清空大脑的认知负担。TaskOS 用 JSONL 中央任务池充当"收集箱"，想到什么随手丢进去，AI 帮你分类、归属、排期，让脑子专注于执行。其中"周回顾"是 GTD 公认维持系统运转的核心仪式，也是 TaskOS 选择周颗粒度的直接依据。

- **心理学支撑**：
  - *自我决定理论*：强调自主性、胜任感和联结感，TaskOS 允许用户按自己节奏推进，通过具体反馈增强胜任感，以诤友式陪伴提供联结。
  - *韧性与恢复*：引入恢复协议和韧性阈值，当系统检测到连续高压或低迷时主动建议降速，保护长期作战能力。
  - *周节律*：研究表明人的精力、注意力和意志力存在周期性波动，以周为单位规划天然贴合这一节律，减少"今天没状态 = 失败"的无谓自责。

方法论搭骨架，以人为中心做内核。

## 使用指南 | Usage Guide

<details open>
<summary><b>🇨🇳 中文（点击切换英文 👇）</b></summary>

### 怎样让 TaskOS 更好地帮到你

TaskOS 像一个记忆力极好的搭档——你告诉它越多，它帮你越准。以下几件事能让合作效果翻倍：

**1. 让它认识你**

刚开始用的时候，花几分钟跟它聊聊：你在忙什么、每周大概有多少空闲时间、哪些事情对你重要。它会建一份画像，之后排计划时就知道什么对你来说是合理的量。

**2. 想到什么随时说**

不需要等到"坐下来整理"的时候。走在路上想到一件事，直接说"记一下：给导师发邮件"就够了。它会帮你归类、排进池子，你的脑子不用记。

**3. 每周花 10 分钟做计划和复盘**

这是让整个系统转起来的关键节奏。周初说"开周计划"，它帮你从池子里挑本周要推的事；周末说"周复盘"，它帮你看看进展、发现卡点。中间的日子随时说"做了 XX"报进度即可。

**4. 状态不好的时候告诉它**

它能采集你的情绪和精力状态。如果这周很累、很焦虑，直接说出来。它会据此调整建议强度，必要时主动建议你降速或进入恢复周。

**5. 允许它拦你**

它有时会拒绝你的请求——比如你想加第 4 个核心项目，或者连续高强度三周还想加量。这是它在保护你。你当然可以坚持，但先听听它的理由。

**6. 把进展说具体**

"背了 20 个单词""读到第 45 页""写了 1500 字"——越具体，它给你的反馈越有力，复盘时的进度感也越真实。

**7. 长期不用也没关系**

两周、一个月没打开，回来时它会帮你平静地重新拾起，陪你从当下开始。

</details>

<details>
<summary><b>🇬🇧 English (click to expand)</b></summary>

### How to get the most out of TaskOS

TaskOS works like a partner with perfect memory — the more you share, the better it helps. Here's how to make the collaboration work well:

**1. Let it get to know you**

Spend a few minutes at the start telling it what you're working on, how much free time you have each week, and what matters to you. It builds a profile so that when it suggests plans, the workload actually fits your life.

**2. Capture thoughts anytime**

No need to wait for a "sit down and organize" session. Walking somewhere and remember something? Just say "capture: email my advisor." It files and categorizes for you — your brain stays free.

**3. Spend 10 minutes weekly on planning and review**

This is the rhythm that keeps the whole system alive. At the start of the week, say "weekly plan" — it helps you pick what to push forward. At the end, say "weekly review" — it shows your progress and surfaces stuck points. In between, just report "did XX" whenever you make progress.

**4. Tell it when you're not doing well**

It tracks your emotional and energy state. If you're exhausted or anxious this week, say so. It adjusts recommendation intensity accordingly, and may proactively suggest slowing down or entering a recovery week.

**5. Let it push back**

Sometimes it will refuse your requests — like adding a 4th core project, or piling on more work after three intense weeks. It's protecting you. You can always override, but hear its reasoning first.

**6. Be specific about progress**

"Memorized 20 words," "read to page 45," "wrote 1500 words" — the more specific, the more powerful its feedback becomes, and the more real your sense of progress during reviews.

**7. It's okay to disappear**

Haven't opened it in two weeks or a month? When you come back, it helps you pick things up calmly and start fresh from where you are now.

</details>

### 更新日志

- **v2.0.0**（2026-07-02）：聚焦长程项目管理（大规模功能删减）
  - **删除 5 个子系统**：许愿卡奖励系统、习惯打卡系统、Gatekeeper 门禁、随手反思、反思增强
  - **保留核心**：三层 SOP（Areas→Projects→Tasks）、项目管理（core≤3 提醒）、风险模型、三档排期（时间制）、周计划/周复盘、心理增强层 5 机制（恢复协议/进展可视化/情绪觉察/决策疲劳防护/价值对齐审计）
  - **架构精简**：SKILL.md 触发词 4 类→删除、强制规则 9 条→8 条、工作流路由 12 条→8 条、journal 标记 19 个→10 个、healthcheck 简化为 13 项核心检查
  - 项目创建改为自由模式（仅保留 core≤3 硬约束提醒，不再做质询）
  - 迁移：废弃文件归档不删除，data_schema 升 2.0.0

- **v1.8.0**（2026-06-02）：Streak 退役 + 启动减负 + bug 修正（减法升级）
  - **许愿卡 Streak 退役**：连续打卡能力存在两套近乎同构的引擎（许愿卡 Streak 与习惯核心层），收敛为一套。连续追踪统一由习惯系统承接，"连续达成攒卡"由悬赏（bounty）承接，"打卡"触发词不再有归属歧义。许愿卡子系统 5→4，重复性事项机制 3→2（习惯/ritual）
  - **启动行为压缩**：从 11 步无条件执行 → 3 步必做硬地板（读 INDEX / 崩溃恢复 / current_week 比对）+ 7 项按需触发（无信号静默跳过），显著降低每次启动的 AI 负担，能力零损失
  - **bug 修正**：① 启动计数校验公式补上 Strategy 段（原只算 Core+Normal+Side 三段，导致有 strategy 项目时每次误报漂移）；② SKILL.md 参考路由"14 项"→"13 项"（与 healthcheck 一致）；③ weekly_est_limit_source 示例统一为 manual
  - 迁移轻量幂等（streaks.md 退役处理：不存在则无操作 / 有数据则询问转习惯或归档，不删数据），data_schema 升 1.8.0

- **v1.7.1**（2026-06-02）：文档中文化 + 默认中文对话 + 核查项修复
  - **默认中文对话**：强制规则新增第 9 条——除非用户当前对话明确要求换语言，否则一律用简体中文交互（仅约束对话语言，不影响数据文件中必须保留的英文）
  - **中英文梳理**：文档可中文化的英文叙述统一中文化（章节标题 `Workflow: Xxx` → `工作流：Xxx`、`Mini-Check` → 即时核查、`Recovery Protocol` → 恢复协议等）；数据锚点（字段名/枚举/ID/文件名/journal 标记/INDEX 段标题）按约定保留英文不变
  - **healthcheck 修复**：全面核查项数全文统一为 13 项，清除历史遗留的"INDEX 当周快照"残留检查
  - 无数据格式变更，data_schema 保持 1.7.0，老数据零迁移

- **v1.7.0**（2026-06-02）：三档排期时间制 + 习惯打卡系统（合并升级）
  - **三档排期：条数制 → 时间制**。周计划 must/should/could 从"限条数"改为"限时间"：保留周总上限（weekly_est_limit，默认 32h）+ 新增 must/should 时间子上限（默认 15h / 建议 8h）+ could 纯弹性不设上限
  - 数字都不写死，首次排期 AI 引导确认后写入 INDEX，用户随时可改；精力档从查死表改为指导层浮动比例
  - 新增 est 缺失三级兜底（推断 / 问询 / 1h），并明确"排期 est 总量"与"业余深度研究参考"是两个不可混用的口径
  - **新增习惯打卡系统**（独立于许愿卡）：核心层（硬上限 3 个，每日批量打卡"默认全过，只报例外"）+ 观察层（不限量，只累计不断签）
  - 旋转门毕业：核心习惯自动化后（~66 天锚点，可拒绝）毕业腾位，仅毕业时发 1 张许愿卡（守"非胡萝卜"哲学）
  - 与许愿卡 Streak / area ritual 按目的划界；新增 workflow-habit.md、habits.md（懒加载，不增启动负担）
  - `core 项目 ≤ 3` 约束保持不变（与三档排期是不同维度）

- **v1.5.1**（2026-05-29）：随手反思
  - 新增 `reflections.md` 滚动文件，可随时记录想法/感受/觉察（触发词：记个反思 / 随手反思 / 想到一件事 / 突然想到）
  - 新增 workflow-reflect.md 工作流，并与任务捕获（capture）做语义区分（"记一下 XX"仍走 capture，向后兼容）
  - 周复盘微反思环节自动汇总本周散记进当周快照 `## 本周反思` 段，形成"即时捕获 → 周度收束"闭环
  - 极简设计：行格式仅 4 字段，独立于任务池不刷 INDEX，刻意不做标签/提醒/即时分析

- **v1.5.0**（2026-05-29）：心理增强层
  - 新增恢复协议（连续高压/低迷时建议恢复周模式）
  - 新增最小可行进展可视化（周复盘展示文本进度条 + progress-update 时对比叙事）
  - 新增情绪觉察层（周计划采集情绪底色，长期模式觉察）
  - 新增反思日志增强（每周微反思问题 + 每月抽象化）
  - 新增决策疲劳防护（AI 预排草案 + 滞留任务决策合批）
  - 新增价值对齐审计（约每季度展示时间投入 vs 声明价值的一致性）
  - 所有新功能为指导层，不增加规则层复杂度
- **v1.4.0**（2026-05-29）：许愿卡奖励系统
  - 引入游戏化正向激励机制：许愿卡流水 + 目标悬赏 + 许愿清单 + 连续纪录 + 每周挑战
  - 系统定位：自我认可仪式 + 允许自己享受的通行证 + 让好事可见的记录本
  - 懒加载设计：不增加启动负载，只在用户触发相关指令时才加载 rewards 文件
  - 周复盘新增许愿卡结算步骤（非阻塞）
  - 周计划完成后 AI 提出本周挑战（可拒绝）
  - 价值锚点：1张 = 一次小确幸的快乐，自然限流防通胀
- **v1.3.0**（2026-05-18）：规则/指导分层重构
  - 正式区分规则层（数据一致性底线）和指导层（AI 判断空间）
  - workflow-weekly.md 从 ~420 行精确脚本重写为 ~200 行分层结构
  - 删除所有固定话术模板，替换为语气指导
  - 阈值从硬编码改为"建议初始值 + 调整原则"
  - AI 获得交互灵活性（可合并问题、调序、自选话术）
  - 强制规则从 9 条精简为 8 条（数据安全底线不变）
- **v1.2.5**（2026-05-18）：双锚点工作量评估（研究基准 + 个人历史校准）
- **v1.2.4**（2026-05-12）：任务周中进度追踪 + INDEX 架构简化
- **v1.2.3**（2026-05-12）：人设增强 + 韧性/探索/模式性阈值机制
- **v1.2.2**（2026-05-12）：新增项目写入门禁 Gatekeeper
- **v1.2.1**（2026-05-11）：路径通用化 + 项目偏好
- **v1.2.0**（2026-05-11）：AI 主动规划能力 + 全维用户画像
- **v1.1.0**（2026-05-11）：精简历史记录 + 复盘能力增强
- **v1.0.0**（2026-05-10）：首版发布
