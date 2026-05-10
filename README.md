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
- `references/` — 详细工作流 SOP（capture / weekly / rename / distill）+ 数据模型
- `templates/` — area / project / JSONL 行模板
- `待优化记录.md` — 已知问题和改进方向

**当前版本**：v0.9.4

---

## 更新日志

### 2026-05-10 首次上传
- 全部 10 个 skill 定义文件上传
- 仓库结构：taskos/ 下完整镜像本地 skill 目录