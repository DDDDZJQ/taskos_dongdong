---
type: project
name: ___                            # 必填，与文件名一致（中文优先）
area: ___                            # 必填，引用某 area.name
priority: normal                     # core | normal | side（注意 core 全局 ≤ 3）
created: 2026-05-10                  # 必填
deadline: 2026-08-10                 # 必填，必须 ≤ created + 3 个月
status: active                       # active | paused | done | dropped
milestone: "（一句话描述里程碑：什么算这个项目完成？）"
progress: 0.0                        # 0-1，用户主观给（首次创建为 0）
risk: 🟢                             # 🟢 | 🟡 | 🔴，由 weekly 自动写入
justification: "___"                 # v1.2.2 必填：通过 Gatekeeper 质询后，简述为什么该项目值得被 TaskOS 管理

# 可选：关键里程碑（防温水煮青蛙）
# 用 key_milestones 列表罗列必须完成的关键节点
# 时间过半但全 pending 时风险模型会强制 🔴
key_milestones:
  - 关键节点 1: pending              # pending | done
  - 关键节点 2: pending
---

# {项目名}

## 项目目标
（这个项目要解决什么问题 / 达成什么结果？）

## 与 area 的关系
（为什么这个项目对「{area}」领域重要？）

## 拆解思路
（从 milestone 反推到具体任务的逻辑）

## 关键决策点
（执行过程中可能要做的关键判断）

## 偏好
（用户对这个项目的个人偏好，AI 做资源推荐和任务拆解时参考。可随时补充。）
（示例：英语项目 → "偏爱美音、沉浸式学习法、不喜欢死记硬背"）
（示例：心理学项目 → "偏好案例导向学习、注重实操练习"）

## 备注
（参考资料 / 灵感 / 风险预判）

---
---

# Strategy Project 模板（v1.2 新增）

> 以下模板用于 `type: strategy` 的长期规划项目。文件命名格式：`[strategy] {目标名}.md`

```yaml
---
type: strategy
name: "[strategy] ___"                    # 必填，[strategy] 前缀 + 目标名
goal: "___"                               # 必填，最终目标一句话描述
target_date: 2028-05                      # 必填，目标达成日期（不受 90 天约束）
created: 2026-05-11                       # 必填
last_reviewed: 2026-05-11                 # 必填，上次检视日期
status: active                            # active | paused | done | dropped
area: ___                                 # 必填，引用某 area.name
priority: core                            # core | normal | side（不占用 core ≤ 3 槽位）
progress: 0.0                             # 按阶段完成数/总阶段数计算（如 2/4=0.50）
risk: 🟢                                  # 不参与风险模型自动计算，仅手动标注
---
```

## 正文结构

```markdown
# [Strategy] {目标名}

## 目标定义

- **最终目标**：
- **为什么**：
- **约束条件**：（可支配时间、预算等）
- **当前基线**：（起点水平）

## 路线图

### 阶段 1：{阶段名}（时间范围）
- **里程碑**：
- **关联 project**：「xxx」「yyy」
- **阶段目标**：

### 阶段 2：{阶段名}（时间范围）
- **里程碑**：
- **关联 project**：
- **阶段目标**：

（按需增加阶段...）

## 资源清单

（按类别分组，每条标注来源和置信度）
- ✅ {推荐内容} — 来源：{URL/书名}，多源验证
- ⚠️ {推荐内容} — 来源：{单一来源}，建议验证
- ❓ {推荐内容} — AI 推测，未找到来源

## 进度检视记录

### {YYYY-MM} 检视
- 当前阶段进度：
- 偏离分析：
- 调整建议：
- 用户决策：

## AI 研究备注

（搜索过程中发现的补充信息，标注日期和来源）
```

## 子 project 追加字段

从 strategy 拆出的普通 project 需在 frontmatter 添加：

```yaml
parent_strategy: "[strategy] xxx"         # 指向父 strategy project 名
strategy_phase: 1                         # 属于路线图第几阶段
```
