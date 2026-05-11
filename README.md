# FOR_AI

AI Agent 的个人技能仓库（private）。

## 目录

- `taskos/` — TaskOS 个人任务管理 Skill

## TaskOS Skill

基于「领域/项目/任务」三层 SOP 的通用个人任务管理 skill。

- 版本：v1.2.1
- 兼容：Claude Code / Cursor / OpenClaw / WorkBuddy 等支持 Anthropic 风格 skill 的 agent
- 数据格式：纯 Markdown + JSONL，跨 AI agent 可移植

### 更新日志

- **v1.2.1**（2026-05-11）：路径通用化 + 项目偏好
  - TASKOS_ROOT 改为 `~/TaskOS`（跨平台通用，不绑定具体用户名）
  - project 模板新增 `## 偏好` 段（记录用户对项目的个人偏好，如英语偏爱美音）
- **v1.2.0**（2026-05-11）：AI 主动规划能力 + 全面用户画像
  - 新增 Nudge 机制：启动时主动扫描停滞/堆积/失衡，给出执行建议
  - 新增 Strategy 工作流：长期目标路线图设计/研究/检视/调整
  - Strategy 作为特殊 project（type: strategy），不占 core 槽位
  - 新增全面用户画像（profile.md，8 大维度）
  - 冷启动"认识你"对话 + 启动询问优先级规则
  - 全面核查 13 项 + 强制规则第 6 条
- **v1.1.0**（2026-05-11）：精简历史记录 + 复盘能力增强 + Skill 更新指引
- **v1.0.0**（2026-05-10）：首版发布