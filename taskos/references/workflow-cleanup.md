# Workflow: Cleanup（数据瘦身）

一键执行数据整理和归档，减少文件体积，保持 TaskOS 目录整洁。**先报告再动手**。

---

## 一、触发词

- 数据瘦身 / 清理文档 / 文档体检 / 收拾收拾

---

## 二、执行流程（7 步）

### 步骤 1：扫描统计

列出 TaskOS 目录下所有文件及行数/大小：

```
TaskOS 文件统计：
├── INDEX.md                    (45 行)
├── .journal.md                 (230 行) ⚠️ 超过 200 行
├── areas/                      (4 个文件)
├── projects/active/            (8 个文件)
├── projects/archive/           (3 个文件)
├── tasks/active.md             (52 行)
├── tasks/inbox.md              (8 行)
├── tasks/done-2026-01.md       (35 行)
├── tasks/done-2026-02.md       (42 行)
├── tasks/done-2026-03.md       (38 行)
├── tasks/done-2026-04.md       (55 行)
├── tasks/done-2026-05.md       (12 行)
├── reviews/                    (12 个 review 文件 + _align-log.md)
├── reviews/_align-log.md       (85 行)
└── reviews/distill/            (4 个文件)
```

### 步骤 2：journal 归档检查

- `.journal.md` 超过 200 行 → 建议按月切分
- 操作：把早于当月的内容切到 `.journal-YYYY-MM.md`
- **注意**：这通常在启动行为中已自动处理；如果没有，此步骤补充执行

### 步骤 3：done-*.md 老化检查

- 超过 6 个月的 done 文件 → 建议压缩归档
- 压缩方式：
  - 保留统计摘要（总完成数、各 project 完成数、各 context 分布）
  - 明细 JSONL 行移入 `tasks/archive/done-YYYY-MM.md`
  - 原位置保留摘要文件（标题 + 元数据 + 统计）
- **例外**：如果该月 done 文件中的数据仍被月度蒸馏引用（最近 3 个月），暂不归档

### 步骤 4：review 文件归档检查

- 超过 12 周的 review 文件 → 确认移入 `reviews/archive/`
- 操作：`mv reviews/2026-W05-review.md reviews/archive/`
- **不删除**，只移动

### 步骤 5：_align-log.md 瘦身

- 超过 100 条记录 → 旧条目归档
- 操作：当年以前的条目移入 `reviews/_align-log-YYYY.md`
- 当年内容保留在 `_align-log.md`

### 步骤 6：INDEX 冗余检查

- 检查 INDEX 是否有已归档 project 仍在列表中
- 检查"上次月度蒸馏"字段格式是否规范
- 如有冗余 → 列入瘦身建议

### 步骤 7：输出瘦身报告

---

## 三、瘦身报告格式

```
📊 数据瘦身建议：

1. journal 归档（节省约 180 行）
   - .journal.md 当前 230 行，建议把 1-4 月内容切到 .journal-2026-01.md ~ .journal-2026-04.md
   - 执行？ [是/否]

2. done 文件归档（节省约 115 行活跃空间）
   - done-2026-01.md（35 行，超过 6 个月）→ 移到 tasks/archive/
   - done-2026-02.md（42 行，超过 6 个月）→ 移到 tasks/archive/
   - done-2026-03.md（38 行，超过 6 个月）→ 移到 tasks/archive/
   - 执行？ [是/否/选择性归档]

3. review 归档（节省 5 个文件）
   - W05 ~ W09 的 review 已超过 12 周 → 移到 reviews/archive/
   - 执行？ [是/否]

4. _align-log 瘦身（节省约 40 行）
   - 当前 85 条，2025 年的 40 条可归档
   - 执行？ [是/否]

5. INDEX 无冗余 ✅

总计可节省：约 335 行活跃空间 + 5 个文件位移
```

---

## 四、关键原则

1. **先报告再动手**：列出所有建议 → 用户逐条确认 → 才执行
2. **只动机器生成的文档**：不动用户手写内容（area.md / project.md 正文部分）
3. **归档而非删除**：旧内容移入 archive/ 子目录，不彻底删除
4. **走 journal**：实际执行文件变更时走 journal（在用户确认后）
5. **不影响数据完整性**：归档后的文件仍可被月度蒸馏和长期趋势查询读取
6. **tasks/archive/ 目录**：用于存放被压缩的老 done 文件（v1.0 新增目录）

---

## 五、与其他工作流的关系

- **启动行为的 journal 月度归档**：是步骤 2 的自动版（每次启动都检查）
- **自动归档规则**（review 12 周 / project 14 天）：是步骤 4 的自动版
- **数据瘦身**是用户主动触发的一次性"大扫除"，覆盖更广、力度更大

---

## 六、对话模板（参考）

### 启动
> 开始数据瘦身扫描...

### 报告
> 扫描完毕，发现 3 处可以整理：
>
> 1. **journal 较长**（230 行）：建议把 1-4 月内容按月归档
> 2. **3 个 done 文件超龄**（2026-01~03）：建议移到 tasks/archive/
> 3. **5 个 review 文件超期**（W05~W09）：建议移到 reviews/archive/
>
> 要执行哪些？可以全部执行，也可以逐条确认。

### 执行中
> 正在执行瘦身操作（已走 journal）...
> - ✅ journal 已按月归档
> - ✅ 3 个 done 文件已移入 archive
> - ✅ 5 个 review 文件已移入 archive
>
> 瘦身完成，节省约 335 行活跃空间。INDEX 已同步更新。

### 无需瘦身
> 扫描完毕，当前数据量健康，暂无需瘦身。
> - journal: 45 行
> - 所有 done 文件都在 6 个月内
> - review 文件都在 12 周内
> - _align-log: 32 条
