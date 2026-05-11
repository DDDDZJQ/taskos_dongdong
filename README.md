# FOR_AI

AI Agent 的个人技能仓库（private）。

## 目录

- `taskos/` — TaskOS 个人任务管理 Skill

## TaskOS Skill

基于「领域/项目/任务」三层 SOP 的通用个人任务管理 skill。

- 版本：v1.2.0
- 兼容：Claude Code / Cursor / OpenClaw / WorkBuddy 等支持 Anthropic 风格 skill 的 agent
- 数据格式：纯 Markdown + JSONL，跨 agent 可移植

### 更新日志

- **v1.2.0**（2026-05-11）：AI 主动规划能力 + 全面用户画像
  - 新增 Nudge 机制：启动时主动扫描停滞/堆积/失衡，给出执行建议
  - 新增 Strategy 工作流：长期目标路线图设计/研究/检视/调整（手动触发，AI 搜索多源交叉验证）
  - Strategy 作为特殊 project（type: strategy），不占 core 槽位，不受 90 天 deadline 约束
  - 新增全面用户画像（profile.md，8 大维度），支撑个性化建议
  - 冷启动"认识你"对话（分 2~3 次自然聊天）
  - 启动询问优先级规则（避免同时轰炸用户）
  - 全面核查从 10 项增至 13 项
  - 新增强制规则：AI 建议不得自动写入
  - 新建 workflow-strategy.md
  - 修改 7 个文件
- **v1.1.0**（2026-05-11）：精简历史记录 + 复盘能力增强 + Skill 更新指引
  - 合并双日志 + 砍掉月度蒸馏预存
  - 结构化周快照 + 决策记录 + 自动最小快照
  - 新增 migration.md 手动触发迁移
- **v1.0.0**（2026-05-10）：首版发布
  - 三层 SOP + JSONL 中央池 + 风险模型 + AI 人格基调