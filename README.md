# FOR_AI

AI Agent 的个人技能仓库（private）。

## 目录

- `taskos/` — TaskOS 个人任务管理 Skill

## TaskOS Skill

基于「领域/项目/任务」三层 SOP 的通用个人任务管理 skill。

- 版本：v1.2.4
- 兼容：Claude Code / Cursor / OpenClaw / WorkBuddy 等支持 Anthropic 风格 skill 的 agent
- 数据格式：纯 Markdown + JSONL，跨 AI agent 可移植

### 更新日志

- **v1.2.4**（2026-05-12）：任务周中进度追踪 + INDEX 架构简化
  - 新增 task 字段：status（not_started/in_progress/blocked）+ note（自由文本进度细节）
  - 新增强制规则第 8 条：用户报告进度时 AI 必须即时写回 active.md
  - 移除 INDEX 当周快照段（一个信息只存一个地方，消除同步问题）
  - 所有任务查询改为直接从 active.md 实时计算
  - 周复盘快照新增 [~] 部分完成 / [!] 阻塞标记
  - 核查清单从 14 项简化为 13 项
- **v1.2.3**（2026-05-12）：人设增强 + 韧性/探索/模式性阈值机制
  - 人设段重写：韧性优先、允许留白、不上纲上线但也不帮找借口
  - 新增韧性检测 + 探索空间提醒 + Strategy 模式性阈值
- **v1.2.2**（2026-05-12）：新增项目写入门禁 Gatekeeper
- **v1.2.1**（2026-05-11）：路径通用化 + 项目偏好
- **v1.2.0**（2026-05-11）：AI 主动规划能力 + 全面用户画像
- **v1.1.0**（2026-05-11）：精简历史记录 + 复盘能力增强
- **v1.0.0**（2026-05-10）：首版发布