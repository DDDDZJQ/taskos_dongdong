# 工作流：全面核查

一键执行数据完整性检查，**纯只读操作**，不走 journal，不修改任何文件。

---

## 一、触发词

- 全面核查 / 系统检查 / 数据体检 / 健康检查

---

## 二、核查清单

| # | 检查项 | 数据源 | 说明 |
|---|--------|--------|------|
| 1 | active.md JSONL 每行合法性 | tasks/active.md | 逐行 JSON.parse，记录坏行位置和内容 |
| 2 | 所有 task 的 projects 引用有效性 | tasks/active.md + projects/active/ + projects/archive/ | projects 数组中每个 project 必须存在于 projects/active/ 或 projects/archive/ |
| 3 | 所有 project 的 area 引用有效性 | projects/active/*.md + areas/*.md | frontmatter.area 必须存在于 areas/ |
| 4 | core 项目数 ≤ 3 | projects/active/*.md | 超出告警（strategy project 不计入） |
| 5 | inbox 积压数量 | tasks/inbox.md | > 20 条提醒清理 |
| 6 | 长期未完成任务列表 | tasks/active.md | created 距今天 > 14 天且 status != "blocked" 的任务 |
| 7 | journal 未配对 [in_progress] | .journal.md | 检查是否有未完成操作 |
| 8 | area.rituals 与 active 中 ritual 任务一致性 | areas/*.md + tasks/active.md | ritual 是否有遗漏/多余 |
| 9 | strategy project 子 project 引用有效性 | projects/active/*.md | 所有 parent_strategy 指向的 strategy project 存在且 active |
| 10 | profile.md 结构完整性 | profile.md | 文件存在、frontmatter 合法（type: user_profile）、completeness 值 0~1 |
| 11 | nudge 冷却段格式合法性 | INDEX.md | 周编号格式正确（YYYY-Www）、无过期冷却（可建议清理） |
| 12 | project justification 完整性 | projects/active/*.md | 所有 active project 应有 justification 字段 |

---

## 三、执行流程

### 步骤 1：读取所有相关文件

按以下顺序读取：
1. INDEX.md
2. tasks/active.md（JSONL 逐行解析）
3. tasks/inbox.md（计数）
4. areas/*.md（列出所有 area 名 + rituals 配置）
5. projects/active/*.md（列出所有 active project 名 + area 引用 + parent_strategy）
6. .journal.md（检查末尾）
7. tasks/done-YYYY-MM.md（当月）
8. profile.md（检查存在性和 frontmatter）

### 步骤 2：逐项检查

按检查清单 #1-#12 逐项执行。每项独立，即使某项检查失败也继续后续检查。

### 步骤 3：汇总输出

---

## 四、输出格式

### 全部通过

```
✅ 全面核查通过，12 项检查全部正常。

详情：
- JSONL 合法行数：47 行，无坏行
- projects 引用全部有效
- area 引用全部有效
- core 项目数：3/3
- inbox 积压：5 条（正常）
- 长期未完成：2 条
- journal 无未配对操作
- ritual 任务一致
- strategy 子 project 引用有效
- profile.md 结构完整
- nudge 冷却段格式正常
- project justification 完整
```

### 有问题

```
⚠️ 全面核查发现 2 项异常：

❌ #2 task 的 projects 引用无效
   - t-20260508-003 引用了不存在的 project「已删除项目」
   - 建议：将该任务 projects 改为 [] 后重新指定归属

⚠️ #6 长期未完成任务：2 条
   - t-20260420-001「整理第一章笔记」创建于 2026-04-20（距今 42 天）
   - t-20260425-003「写播客大纲」创建于 2026-04-25（距今 37 天）
   - 建议：考虑推进或放弃

其余 10 项检查正常。

是否需要修复上述问题？（修复会走 journal）
```

---

## 五、关键原则

1. **纯只读**：不写任何文件，不动 journal，不动 INDEX
2. **不擅自修复**：发现问题只报告 + 给修复建议 + 询问用户
3. **不主动重建 INDEX**：只有检查发现问题时才询问是否重建
4. **用户确认后修复走 journal**：如果用户说"修复"，则进入写操作流程，此时走 journal
5. **ritual_source 不参与孤儿检查**

---

## 六、对话模板（参考）

### 启动
> 开始全面核查（12 项检查）...

### 完成（有问题）
> 核查完毕，发现 2 处需要注意：
> 1. 有 1 条任务引用了不存在的 project「已删除项目」
> 2. 有 2 条任务创建超过 14 天未推进，建议处理
>
> 要我帮你修复吗？

### 完成（无问题）
> ✅ 全面核查通过，12 项全部正常。数据状态健康。
