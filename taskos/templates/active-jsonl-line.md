# `tasks/active.md` JSONL 行格式范例

本模板说明 `tasks/active.md`（中央任务池）和 `tasks/done-YYYY-MM.md`（已完成 / 放弃归档）中 JSONL 区块每一行的标准写法。

---

## 一、文件整体结构

```markdown
# Active Tasks
## ~~~ JSONL 区块开始 ~~~
（每行一个 JSON 对象，详见下方示例）
## ~~~ JSONL 区块结束 ~~~
```

> done-YYYY-MM.md 标题为 `# Done Tasks YYYY-MM`，其余结构相同。

---

## 二、active.md 标准任务示例

### 2.1 普通任务，挂某 project

```json
{"id":"t-20260510-001","title":"精读第 3 章","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@阅读","est":"90min","captured":"2026-05-10","status":"not_started","note":null}
```

### 2.2 普通任务，仅挂 area（无具体 project）

```json
{"id":"t-20260510-007","title":"联系张老师确认督导","projects":[],"area":"心理学","context":"@电话","est":"15min","captured":"2026-05-10","status":"not_started","note":null}
```

### 2.3 普通任务，多挂（同时推进多个 project）

```json
{"id":"t-20260510-003","title":"写一篇关于客体关系的文章","projects":["精读《动力取向心理治疗》","自媒体第 5 期录制"],"area":null,"context":"@写作","est":"180min","captured":"2026-05-10","status":"not_started","note":null}
```

### 2.4 周频 ritual 任务

```json
{"id":"r-2026-W19-001","title":"精读 1 篇外刊封面文章","projects":[],"area":"心理学","context":"@阅读","est":"60min","ritual_source":"心理学","ritual_desc":"精读 1 篇外刊封面文章","freq":"weekly","captured":"2026-05-10","status":"not_started","note":null}
```

### 2.5 月频 ritual 任务

```json
{"id":"r-2026-05-001","title":"听完 5 期专业播客","projects":[],"area":"心理学","context":"@阅读","est":"300min","ritual_source":"心理学","ritual_desc":"听完 5 期专业播客","freq":"monthly","captured":"2026-05-04","status":"not_started","note":null}
```

### 2.6 未处理的任务（pool 状态）

```json
{"id":"t-20260428-003","title":"整理第一章笔记","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@电脑","est":"60min","status":"not_started","note":null,"captured":"2026-04-28"}
```

> v2.1.0 删除了 due_week / tier / carry 字段。任务只在 active.md 池中，不再分配到周。

### 2.7 进行中任务（含 status + note）

```json
{"id":"t-20260512-001","title":"背 200 单词","projects":["英语学习"],"area":null,"context":"@阅读","est":"60min","status":"in_progress","note":"110/200，Unit 5-6 已完成，剩 Unit 7-8","captured":"2026-05-12"}
```

> 用户报告进度后 AI 即时更新 status + note。

### 2.8 阻塞任务

```json
{"id":"t-20260512-003","title":"提交周报","projects":["工作"],"area":null,"context":"@电脑","est":"30min","status":"blocked","note":"等主管确认新模板","captured":"2026-05-12"}
```

---

## 三、done-YYYY-MM.md 标准任务示例

done 文件保留 active 时**全部字段（含 null）** + `completed` + `outcome` + `week`。

### 3.1 已完成

```json
{"id":"t-20260509-005","title":"写文章大纲","projects":["自媒体第 5 期录制"],"area":null,"context":"@写作","est":"60min","captured":"2026-05-08","status":"in_progress","note":"大纲已完成","completed":"2026-05-09","outcome":"done","week":"2026-W18"}
```

### 3.2 已放弃

```json
{"id":"t-20260428-003","title":"整理第一章笔记","projects":["精读《动力取向心理治疗》"],"area":null,"context":"@电脑","est":"60min","captured":"2026-04-28","status":"not_started","note":null,"completed":"2026-05-10","outcome":"dropped","week":null}
```

### 3.3 已完成的 ritual

```json
{"id":"r-2026-W18-001","title":"精读 1 篇外刊封面文章","projects":[],"area":"心理学","context":"@阅读","est":"60min","ritual_source":"心理学","ritual_desc":"精读 1 篇外刊封面文章","freq":"weekly","captured":"2026-05-04","status":"not_started","note":null,"completed":"2026-05-08","outcome":"done","week":"2026-W18"}
```

---

## 四、字段填法约定

| 字段 | 填法 |
|---|---|
| `id` | 严格按 ID 编码规范（schema.md 第六节） |
| `title` | 动词前置改写后的描述（capture 时保证） |
| `projects` | 数组；空数组 `[]` 不省略 |
| `area` | 仅挂 area 时填；否则 null |
| `context` | `@阅读` / `@电脑` / `@电话` / `@外出` / `@写作` 等；不确定填 null |
| `est` | 估时字符串，如 `"90min"` / `"2h"`；不确定填 null |
| `ritual_source` | 仅 ritual 任务有，值 = 来源 area 名 |
| `ritual_desc` | 仅 ritual 任务有，值 = 来源 area 中 rituals 数组的 desc 文本 |
| `freq` | 仅 ritual 任务有，值 = `"weekly"` / `"monthly"` |
| `captured` | YYYY-MM-DD 格式 |
| `status` | `"not_started"` / `"in_progress"` / `"blocked"`；默认 `"not_started"` |
| `note` | 自由文本字符串，或 null；每次进度更新覆盖旧值 |
| `completed` | 仅 done 文件有，YYYY-MM-DD |
| `outcome` | 仅 done 文件有，`"done"` / `"dropped"` |
| `week` | 仅 done 文件有，ISO 周字符串或 null |

**v2.1.0 删除字段**：`carry`、`due_week`、`tier`

---

## 五、JSONL 一行的"完整性"自检

每次 AI 写一行前，自查以下：

1. ✅ 双引号包裹所有字符串
2. ✅ null 值小写 `null`
3. ✅ 空数组 `[]`
4. ✅ 必填字段都填了（id / title / projects / captured；done 文件再加 completed / outcome）
5. ✅ id 全局唯一（在 active + 当 captured 月 done + inbox 中扫描）
6. ✅ 整行占满一行不跨行

通过自检后再写入文件。

---

## 六、错误示例（不要这样写）

### ❌ 缺少必填字段
```json
{"id":"t-20260510-001","title":"精读第 3 章"}
```

### ❌ 用单引号 / Python 风格
```json
{'id':'t-20260510-001'}
```

### ❌ null 大写
```json
{"status":"NULL"}
```

### ❌ 空数组写成 null
```json
{"projects":null}
```
正确应为 `"projects":[]`。

### ❌ 跨行
```json
{
  "id": "t-20260510-001",
  "title": "..."
}
```
JSONL 必须单行。
