# 工作流：重命名

负责 area / project 的重命名操作，确保所有引用同步更新，同时保留历史 done-*.md 的旧名以维持历史真相。

---

## 一、触发词

- 把 X 改名为 Y
- 把 area X 改名为 Y
- 把项目 X 改名为 Y
- X 改名为 Y / X 改成 Y / 重命名 X

---

## 二、操作流程

### 步骤 1：判断重命名对象类型

- 如果 X 在 `areas/` 下存在 → area 重命名
- 如果 X 在 `projects/active/` 或 `projects/archive/` 下存在 → project 重命名
- 如果都找不到 → 提示用户"X 不存在，请确认名称"

### 步骤 2：journal [in_progress]

记录意图。

```markdown
[2026-05-10 05:18] #287 in_progress | rename area「心理学」→「临床心理学」
```

### 步骤 3：执行替换

#### 3.1 改文件名 + frontmatter.name
- areas 重命名：`areas/X.md` → `areas/Y.md`，文件内 `name: Y`
- project 重命名：`projects/active/X.md` → `projects/active/Y.md`，文件内 `name: Y`

#### 3.2 全局替换引用

**对 area 重命名**：
- `projects/active/*.md` 和 `projects/archive/*.md` 的 frontmatter `area: X` → `area: Y`
- `tasks/active.md` 的 JSONL 行：`"area":"X"` → `"area":"Y"`，`"ritual_source":"X"` → `"ritual_source":"Y"`
- `tasks/inbox.md` 中如有引用文本，可酌情更新（inbox 阶段是脑暴草稿，引用通常是隐式的）

**对 project 重命名**：
- `tasks/active.md` 的 JSONL 行：`"projects":["X", ...]` → `"projects":["Y", ...]`（保留数组中其他元素）

#### 3.3 INDEX 全部引用更新

- areas 列表 / projects 表中的名称
- **当前周快照中"→ X"引用**（如"- [ ] t-20260510-001 精读第 3 章 → X" → "→ Y"）
- by_project_count 中的 key

### 步骤 4：历史 done-*.md **不改**

> 保留历史真相："X 在被改名前，曾叫 X"。

历史 done-*.md 中的引用（projects 数组、area、ritual_source）保持不变。

### 步骤 5：写改名记录到 .journal.md

```markdown
[2026-05-10 05:18] #288 align | rename area「心理学」→「临床心理学」, done-*.md 保留旧名
```

### 步骤 6：journal [done] + 即时核查 + INDEX 刷新

```markdown
[2026-05-10 05:18] #287 done | rename area「心理学」→「临床心理学」
```

---

## 三、约束与规则

### 仅改 name 相关字段
重命名**只改 name 字段及其引用**。其他字段（priority / status / area / progress / risk / created / deadline / milestone / key_milestones / yearly_intent / rituals / identity）**保持不变**。

### 不允许改名为已存在的名字
- 如果用户说"把 area X 改名为 Y"，但 areas/ 下已有 Y.md → 拒绝并提示"Y 已存在，请换个名字或先合并"
- 同理 project 重命名

### 月度蒸馏的统计合并
复盘工作流（workflow-retrospect.md）在实时生成分析时，必须读 journal 中的 `[align]` 改名记录，把"X 在 YYYY-MM-DD 改名为 Y"视为同一对象。

### 不支持区域跨级重命名
- 不能把 area 改名为 project，也不能反过来。
- 不能把 X 从 area 移动到 project（这是结构变更，不是改名）。

---

## 四、即时核查（rename 后立即跑）

| 检查项 | 说明 |
|---|---|
| 全局检查旧名是否还有残留（除 done-*.md 外） | 用 grep / 全文搜索旧名 X，应只在 done-*.md 和 .journal.md 中存在 |
| INDEX 同步 | areas 列表 / projects 表 / 当前周快照"→ X"引用 / by_project_count 都应为 Y |
| journal 是否记一笔 [align] | 必须有改名记录 |
| 其他字段未被误改 | priority / status / area / 等保持原值 |

不一致 → 立即提示用户 + 给修复建议（不擅自修复）。

---

## 五、AI 与用户对话模板（参考）

### 重命名前确认
> 准备把「心理学」改名为「临床心理学」。这会影响以下文件：
> - areas/心理学.md → areas/临床心理学.md
> - 3 个 active project 的 area 字段
> - 1 个 archive project 的 area 字段
> - active.md 中 5 条 task 的 area / ritual_source 字段
> - INDEX.md 各处引用
> - 历史 done-*.md 中保留旧名"心理学"以维持历史真相
>
> 确认执行？

### 重命名后总结
> 已完成重命名。
> - 改了 N 个文件
> - 历史 done-*.md 保留旧名（N 条记录引用"心理学"）
> - _align-log.md 已记录改名历史
> - 月度蒸馏统计时会自动合并
