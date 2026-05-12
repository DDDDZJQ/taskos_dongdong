# Workflow: Healthcheck（全面核查）

一键执行 14 项数据完整性检查，**纯只读操作**，不走 journal，不修改任何文件。

---

## 一、触发词

- 全面核查 / 系统检查 / 数据体检 / 健康检查

---

## 二、核查清单（14 项）

| # | 检查项 | 数据源 | 说明 |
|---|--------|--------|------|
| 1 | active.md JSONL 每行合法性 | tasks/active.md | 逐行 JSON.parse，记录坏行位置和内容 |
| 2 | 所有 task 的 projects 引用有效性 | tasks/active.md + projects/active/ + projects/archive/ | projects 数组中每个 project 必须存在于 projects/active/ 或 projects/archive/ |
| 3 | 所有 project 的 area 引用有效性 | projects/active/*.md + areas/*.md | frontmatter.area 必须存在于 areas/ |
| 4 | INDEX 当周快照 vs active.md 过滤一致性 | INDEX.md + tasks/active.md | 按 ID 集合比较（未完成段 + 已完成段） |
| 5 | core 项目数 ≤ 3 | projects/active/*.md | 超出告警（strategy project 不计入） |
| 6 | inbox 积压数量 | tasks/inbox.md | > 20 条提醒清理 |
| 7 | 滞留任务列表 | tasks/active.md + INDEX.current_week | due_week 过期且未完成的任务（把 ISO 周转日期比较） |
| 8 | carry ≥ 3 任务列表 | tasks/active.md | 结转过多的任务 |
| 9 | journal 未配对 [in_progress] | .journal.md | 检查是否有未完成操作 |
| 10 | area.rituals 与 active 中 ritual 任务一致性 | areas/*.md + tasks/active.md | ritual 是否有遗漏/多余（本周应生成但未生成的） |
| 11 | strategy project 子 project 引用有效性 | projects/active/*.md | 所有 parent_strategy 指向的 strategy project 存在且 active |
| 12 | profile.md 结构完整性 | profile.md | 文件存在、frontmatter 合法（type: user_profile）、completeness 值 0~1 |
| 13 | nudge 冷却段格式合法性 | INDEX.md | 周编号格式正确（YYYY-Www）、无过期冷却（可建议清理） |
| 14 | project justification 完整性 | projects/active/*.md | 所有 active project 应有 justification 字段（存量审查前的旧项目豁免） |

---

## 三、执行流程

### 步骤 1：读取所有相关文件

按以下顺序读取：
1. INDEX.md（获取 current_week、active 总数等基准数据）
2. tasks/active.md（JSONL 逐行解析）
3. tasks/inbox.md（计数）
4. areas/*.md（列出所有 area 名 + rituals 配置）
5. projects/active/*.md（列出所有 active project 名 + area 引用 + parent_strategy）
6. .journal.md（检查末尾）
7. tasks/done-YYYY-MM.md（当月，用于已完成段一致性检查）
8. profile.md（检查存在性和 frontmatter）

### 步骤 2：逐项检查

按检查清单 #1-#10 逐项执行。每项独立，即使某项检查失败也继续后续检查。

### 步骤 3：汇总输出

---

## 四、输出格式

### 全部通过

```
✅ 全面核查通过，14 项检查全部正常。

详情：
- JSONL 合法行数：47 行，无坏行
- projects 引用全部有效
- area 引用全部有效
- INDEX 当周快照与 active.md 一致
- core 项目数：3/3
- inbox 积压：5 条（正常）
- 滞留任务：0 条
- carry ≥3 任务：0 条
- journal 无未配对操作
- ritual 任务一致
```

### 有问题

```
⚠️ 全面核查发现 2 项异常：

❌ #4 INDEX 当周快照与 active.md 不一致
   - INDEX 中有但 active.md 中没有：t-20260505-002
   - active.md 中有但 INDEX 中没有：t-20260508-003
   - 建议：重新生成 INDEX 当周快照段

⚠️ #8 carry ≥3 任务：2 条
   - t-20260420-001「整理第一章笔记」carry=4
   - t-20260425-003「写播客大纲」carry=3
   - 建议：考虑拆小或放弃

其余 12 项检查正常。

是否需要修复上述问题？（修复会走 journal）
```

### 发现严重不一致

如果 INDEX 多处漂移（如 #4 + area/project 数量也不对）：
```
🔴 INDEX 存在严重漂移（多处不一致），建议全量重建 INDEX。
是否立即执行 INDEX 全量重建？
```

---

## 五、关键原则

1. **纯只读**：不写任何文件，不动 journal，不动 INDEX
2. **不擅自修复**：发现问题只报告 + 给修复建议 + 询问用户
3. **不主动重建 INDEX**：只有检查发现问题时才询问是否重建
4. **用户确认后修复走 journal**：如果用户说"修复"，则进入写操作流程，此时走 journal
5. **ritual_source 不参与孤儿检查**：与完整性扫描规则一致

---

## 六、与其他工作流的关系

- **Weekly Review 的完整性扫描**是 4 项子集（#2、#3、#4 + INDEX 未列入检查）
- **全面核查**是 14 项完整版，覆盖更广
- 用户随时可以调用全面核查，不需要等周复盘
- 如果刚做完周复盘就做全面核查，大概率全部通过（因为 review 已做了 INDEX 重建）

---

## 七、对话模板（参考）

### 启动
> 开始全面核查（14 项检查）...

### 完成（有问题）
> 核查完毕，发现 2 处需要注意：
> 1. INDEX 当周快照对不上（有 1 条任务多余/缺失）
> 2. 有 1 条任务引用了不存在的 project「已删除项目」
>
> 要我帮你修复吗？修复内容：
> - 重新生成 INDEX 当周快照段
> - 将该任务的 projects 改为 []（等你重新指定归属）

### 完成（无问题）
> ✅ 全面核查通过，14 项全部正常。数据状态健康。
