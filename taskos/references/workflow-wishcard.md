# 工作流：许愿卡奖励系统

---

## 触发词

```
- "许愿卡" / "奖励自己" / "给自己发卡" / "消耗许愿卡" / "兑换"
- "设悬赏" / "完成悬赏" / "目标悬赏"
- "许愿想xxx" / "加个奖励" / "奖励清单" / "看看许愿清单"
- "本周挑战" / "接受挑战" / "拒绝挑战" / "挑战完成了"
- "许愿卡历史" / "奖励记录" / "余额多少"
```

> 连续打卡能力已于 v1.8.0 退役（原 Streak 子系统），统一由习惯系统（workflow-habit.md）承接。"打卡"触发词路由到习惯系统，不再进入许愿卡。

---

## 系统定位（AI 必须理解）

许愿卡是**自我认可仪式 + 许可通行证 + 可见化记录本**。

- ❌ 不是激励做困难事的胡萝卜
- ❌ 不是考勤/KPI
- AI 不催促发卡、不追踪频率、不暗示"你最近发卡少了"
- 用户说奖励就发，AI 不评判理由是否"够格"

---

## ⚙️ 前置检查

所有许愿卡操作前：
1. 检查 `${TASKOS_ROOT}/rewards/` 目录是否存在
   - 不存在 → 提示初始化（详见下方"初始化"段）
2. 读 INDEX.md 的 `## Wish Cards` 段获取余额

---

## 操作路由

| 用户意图 | 操作 | 读取文件 |
|----------|------|----------|
| 奖励自己 N 张 | → 直接获取 | ledger.md |
| 消耗 N 张 / 兑换 xxx | → 消耗 | ledger.md (+wishlist.md) |
| 设悬赏 | → 创建悬赏 | bounties.md |
| 完成悬赏 | → 兑现悬赏 | bounties.md + ledger.md |
| 许愿想 xxx | → 添加 wishlist | wishlist.md |
| 接受/拒绝挑战 | → 挑战响应 | challenges.md |
| 挑战完成了 | → 完成挑战 | challenges.md + ledger.md |
| 余额 / 历史 | → 查询 | ledger.md |
| 看看许愿清单 | → 展示 | wishlist.md |

---

## 一、直接获取

### 触发

用户说"奖励自己 N 张许愿卡，因为 xxx" 或等价表述。

### SOP

1. 读 ledger.md
2. 生成新 ID：`wc-YYYYMMDD-NNN`（扫描当天已有 ID 确保唯一）
3. append 新行：
   ```json
   {"id":"wc-...","type":"earn","amount":N,"reason":"xxx","source":"direct","date":"YYYY-MM-DD"}
   ```
4. 更新 INDEX：`wish_cards_balance += N`，`wish_cards_earned_total += N`
5. journal：`[时间] #op done | wish-card-earn +N "xxx"`
6. 回复确认

### 💡 AI 主动提议

当用户报告了一件值得鼓励的事但没主动要卡时，AI 可以温和提议：
- "这件事值得奖励自己，要不要给自己发 1 张许愿卡？"
- 用户说不用 → 不追问

---

## 二、消耗

### 触发

用户说"消耗 N 张许愿卡，奖励自己 xxx" 或 "兑换 xxx"。

### SOP

1. 检查余额：`INDEX.wish_cards_balance >= N`
   - 不足 → 温和提醒："目前余额 X 张，不够 N 张。要部分兑换还是先攒攒？"
   - 充足 → 继续
2. 如果用户说"兑换"某条 wishlist 中的条目 → 读 wishlist.md 定位条目 + 获取价格
3. append ledger 新行：
   ```json
   {"id":"wc-...","type":"spend","amount":N,"reason":"xxx","date":"YYYY-MM-DD"}
   ```
4. 如从 wishlist 兑换 → 将该条目 `- [ ]` 改为 `- [x]`
5. 更新 INDEX：`wish_cards_balance -= N`，`wish_cards_spent_total += N`
6. journal：`[时间] #op done | wish-card-spend -N "xxx"`
7. 回复确认（带点仪式感："许愿达成！享受吧。"）

---

## 三、悬赏管理

### 创建悬赏

1. 用户说"设悬赏：完成 xxx 奖励 N 张"
2. 读 bounties.md
3. 生成 ID：`b-YYYYMMDD-NNN`
4. 确定 auto_detect：
   - 用户明确说了检测条件 → 写入
   - 否则 → null（手动声明完成）
5. append 新行
6. journal：`[时间] #op done | bounty-create "xxx" reward:N`

### 兑现悬赏

1. 用户说"完成悬赏了" 或 AI 在周复盘中检测到
2. 读 bounties.md 找到对应悬赏
3. 更新悬赏：`status → "completed"`，`completed → 今天`
4. 自动触发 earn（source: "bounty"，bounty_id 关联）
5. 更新 INDEX balance
6. journal：`[时间] #op done | bounty-complete "xxx" +N`

---

## 四、习惯毕业奖励（v1.7.0，与习惯系统联动）

习惯打卡系统（workflow-habit.md）独立运作，但有**唯一一个**与许愿卡的接触点：**核心习惯毕业时发 1 张许愿卡**作里程碑庆祝。

- 触发：核心层习惯毕业（status → graduated，详见 workflow-habit.md §七）
- 动作：自动 earn 1 张，`source: "habit-graduate"`，reason 写习惯名（如"『喝水』养成毕业"）
- ledger 行示例：`{"id":"wc-...","type":"earn","amount":1,"reason":"『喝水』养成毕业","source":"habit-graduate","date":"YYYY-MM-DD"}`
- journal：`[时间] #op done | wish-card-earn +1 "habit-graduate:喝水"`
- **rewards/ 目录不存在**（用户没用许愿卡）→ 毕业时跳过发卡，仅文字祝贺，不阻塞
- **日常习惯打卡不发卡**（守"非胡萝卜"哲学，避免外部奖励侵蚀内在动机）

> earn source 枚举（v1.8.0）：`direct` / `bounty` / `challenge` / `habit-graduate`
>
> **v1.8.0 变更**：原 Streak 连续纪录子系统已退役。想追踪"连续达成攒卡"的需求，用悬赏（bounty）承接——设一个"连续 X 天完成 Y → 奖励 N 张"的悬赏，达成后手动声明完成发卡。连续天数本身由习惯系统的核心层 `streak` 字段追踪。

---

## 五、许愿清单

### 添加

1. 用户说"许愿想 xxx，需要 N 张"
2. 读 wishlist.md
3. 按 N 的大小归类到对应分组（1~2 / 3~5 / 8+）
   - 分组不存在 → 创建分组标题
4. 在对应分组下 append：`- [ ] xxx — N 张`
5. journal：`[时间] #op done | wishlist-add "xxx" cost:N`

### 查看

用户说"看看许愿清单" → 读 wishlist.md 并展示（标注余额是否够兑换每一项）

---

## 六、每周挑战

### 提出（AI 主动，在周计划完成后）

详见 workflow-weekly.md。

### 接受/拒绝

1. 用户回应挑战提议
2. 读 challenges.md 找到本周挑战
3. 更新：status → "accepted"/"declined"，responded → 今天
4. journal：`[时间] #op done | challenge-accept/decline`

### 完成

1. 用户说"本周挑战完成了"
2. 读 challenges.md 确认本周有 accepted 挑战
3. 更新：status → "completed"，completed → 今天
4. 自动触发 earn（source: "challenge"）
5. 更新 INDEX
6. journal：`[时间] #op done | challenge-complete +N`

---

## 七、历史查询

### 触发

"许愿卡历史" / "奖励记录" / "余额多少" / "这个月获得了多少"

### SOP

1. 读 ledger.md
2. 按用户请求范围统计（本月/本周/全部）
3. 展示格式（灵活调整）：

```
许愿卡月报（YYYY 年 M 月）

余额：X 张

本月获取：+N 张
  直接获取    ×A（具体原因列举）
  完成悬赏    ×B
  每周挑战    ×C
  习惯毕业    ×D

本月消耗：-M 张
  具体消耗列举
```

> 习惯连续天数查询走习惯系统（"我的习惯"，见 workflow-habit.md），不在许愿卡月报中。

---

## 八、初始化

首次触发许愿卡相关指令且 `${TASKOS_ROOT}/rewards/` 不存在时：

```
检测到许愿卡系统还没初始化。要不要创建？

这会新增：
- rewards/ledger.md（流水账）
- rewards/bounties.md（目标悬赏）
- rewards/wishlist.md（许愿清单，初始为空）
- rewards/challenges.md（每周挑战）
- INDEX.md 新增 Wish Cards 段（余额 0）

初始化吗？
```

用户同意后创建：

```markdown
# Wish Cards Ledger
## ~~~ JSONL 区块开始 ~~~
## ~~~ JSONL 区块结束 ~~~
```

（bounties/challenges 同理）

wishlist 初始化为：
```markdown
# Wish List

（说"许愿想xxx，需要N张许愿卡"来添加条目）
```

INDEX 新增：
```yaml
## Wish Cards
wish_cards_balance: 0
wish_cards_earned_total: 0
wish_cards_spent_total: 0
current_challenge: null
challenge_status: null
```

journal：`[时间] #op done | wish-card-system-init`

---

## ⚙️ 规则层（不可违反）

1. **余额一致性** — INDEX.wish_cards_balance = ledger earn 总和 - spend 总和
2. **不允许透支** — spend 前检查 balance ≥ amount
3. **流水 append-only** — 不删除记录；修正用 `type: "adjust"`
4. **所有写操作走 journal**
5. **整数单位** — amount 只接受正整数
6. **ID 唯一性** — 各类 ID 在各自文件内全局唯一
7. **懒加载** — rewards 文件只在用户触发或周复盘时读取

---

## 💡 指导层（AI 灵活判断）

- 积极鼓励但不催促；消耗不评判
- 系统存在感要低——用户不提不主动推销
- 允许系统休眠——两周不用不提醒
- 警惕过度理由效应——为了得卡而做事时温和指出
- 价值锚点：1 张 ≈ 一杯奶茶的快乐
- 每日直接获取建议 ≤ 2 张（不强制）
