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

## 备注
（参考资料 / 灵感 / 风险预判）
