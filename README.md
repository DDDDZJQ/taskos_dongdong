# FOR_AI

AI skill 定义文件备份仓库（私有）。

---

## taskos/

**TaskOS** — 通用个人任务管理 skill

基于「领域 (Areas) → 项目 (Projects) → 任务 (Tasks)」三层结构，帮助用户管理长期目标和每周行动。

**核心特点**：
- 任务永不丢失（JSONL 中央池）
- 风险驱动（自动计算项目健康度）
- 懒人友好（两周不开也能温和重启）
- 跨 AI agent 可移植（不依赖私有工具）

**文件结构**：
- `SKILL.md` — 主入口，启动行为 + 工作流路由 + 强制规则
- `references/` — 详细工作流 SOP（capture / weekly / rename / retrospect / healthcheck / cleanup / migration）+ 数据模型
- `templates/` — area / project / JSONL 行模板

**当前版本**：v1.1.0

---

## 更新日志

### v1.1.0 (2026-05-11)
- **精简历史记录**：合并双日志（journal 吸收 _align-log），journal 改为单行格式
- **砍掉月度蒸馏预存**：删除 workflow-distill.md 和 reviews/distill/ 目录，改为按需实时生成
- **去掉 active.md/done-*.md 顶部统计**：AI 按需计数，减少写放大
- **复盘能力增强**：周快照文件改为纯结构化数据（项目状态 + 上下文分布 + carry 趋势）
- **自动最小快照**：current_week 切换时自动保存上周快照，保证时间线连续
- **决策记录**：journal 新增 [decision] 标记，可追溯关键决策
- **手动复盘工作流**：新增 workflow-retrospect.md（替代 distill，从快照实时生成趋势分析）
- **Skill 更新迁移指引**：新增 migration.md，手动触发数据迁移
- 新建：workflow-retrospect.md / migration.md
- 删除：workflow-distill.md
- 修改：SKILL.md / schema.md / workflow-weekly.md / workflow-capture.md / workflow-rename.md / workflow-healthcheck.md / workflow-cleanup.md / active-jsonl-line.md

### v1.0.0 (2026-05-10)
- **AI 人格基调**：SKILL.md 加入诤友式陪伴性格描述
- **精力管理**：weekly plan 新增精力状态询问，动态调整排期上限
- **工作量监控**：排期后自动估算总工时，超限+留白提醒
- **复盘先看成就**：review 开头先列本周完成 task
- **长期历史趋势**：展示近 N 周完成量/carry/core 推进/精力趋势
- **全面核查**：新增一键指令（11 项检查，纯只读）
- **数据瘦身**：新增一键指令（7 步流程，先报告再动手）
- **精力分布统计**：月度蒸馏新增精力-产出对照
- 新建：workflow-healthcheck.md / workflow-cleanup.md
- 修改：SKILL.md / schema.md / workflow-weekly.md / workflow-distill.md

### 2026-05-10 首次上传
- 全部 10 个 skill 定义文件上传
- 仓库结构：taskos/ 下完整镜像本地 skill 目录