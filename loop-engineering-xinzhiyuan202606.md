# Claude Code 之父删了 IDE！干掉提示词，只写循环

> **来源：** 新智元（Synced）  
> **日期：** 2026 年 6 月  
> **原文：** https://mp.weixin.qq.com/s/-_CStmAkQXZBTqpVO17yog  
> **归档日期：** 2026-06-21

---

## 摘要

2026 年 6 月第一周，"Loop Engineering（循环工程）" 成为全球 AI 编程圈最热话题。Claude Code 之父 Boris Cherny 删掉 IDE 只用循环驱动编码；OpenClaw 创始人 Peter Steinberger 的推文 24 小时破 500 万浏览；Google 工程主管 Addy Osmani 撰文定名。本文梳理了这场范式迁移的来龙去脉、实操方法和挑战。

---

## 一、引爆点：三个人的三件事

### Boris Cherny（Claude Code 之父）

- 2025 年 11 月：亲手卸载 IDE，一个月提 259 个 PR，全由 Claude Code 写出
- 2026 年 6 月 2 日访谈：

> *"我不再给 Claude 写提示词了：是循环在跑，是它们在提示 Claude。我的工作就是写循环。"*

### Peter Steinberger（OpenClaw 创始人，OpenAI）

- 2026 年 6 月 7 日 X 推文，24 小时内 500 万浏览：

> *"别再给编程智能体写提示词了，你该去设计那些替你提示智能体的循环。"*

- 推文地址：https://x.com/steipete/status/2063697162748260627

### Addy Osmani（Google 工程主管，前 Chrome 开发者体验负责人）

- 2026 年 6 月 8 日发布博客文章 **《Loop Engineering》**
- 正式为这一概念命名并提出五/六大构件体系
- 文章地址：https://addyosmani.com/blog/loop-engineering/

---

## 二、什么是循环工程

### 过去：回合制

```
人→敲提示词→等AI回复→人→再敲→再等...（人就是那个循环本身）
```

### 现在：产线制

```
人→设计循环→循环自动调度AI→检查完成→记录进度→决定下一步
```

**Addy Osmani 的定义：**

> *"循环工程就是把那个负责提示智能体的人替换掉，你转而去设计那套替你提示的系统。循环本身是一个递归式的目标：你定下目的，让 AI 一轮轮迭代，直到完成。"*

**The New Stack 的类比：**

> 从「操作机床」变成「设计机床所在的整条产线」。

---

## 三、循环的六个构件

| 构件 | 功能 | 工具 |
|------|------|------|
| **Automation**（自动化） | 定时触发，自己去做发现和分诊 | Claude Code Routines / Codex Automations |
| **Worktrees**（隔离工作区） | 防打架，每个子智能体独立代码检出 | git worktree |
| **Skills**（技能） | 把项目惯例、踩坑经验写进 SKILL.md，治金鱼记忆 | 项目级 CLAUDE.md / SKILL.md |
| **Connectors**（连接器） | 基于 MCP，读 issue、查数据库、往 Slack 发消息 | MCP 协议 |
| **Sub-agents**（子智能体） | 写查分离。写代码的不许给自己打分，另开模型挑刺 | /batch + 动态工作流 |
| **Memory**（记忆） | 一个 md 文件，记下做了什么、过了什么、还剩什么 | CLAUDE.md 蒸馏 |

---

## 四、实操案例：@Av1dlive 的「THE HIVE」

开发者 @Av1dlive 根据 Boris 的公开访谈，从零重建了一套三层系统：

### 第一层：本地循环（/loop）

- 靠 `/loop`，开着会话时持续干活
- 最小 1 分钟跑一次
- 关掉电脑就停

### 第二层：云端例程（Routines）

- Anthropic 2026 年 4 月推出
- 跑在云上、新克隆的仓库里
- 关电脑照样跑，最小 1 小时一次

### 第三层：集群（/batch + 动态工作流）

- 一次性把活扇出去
- 几百上千个 worktree 子智能体并行
- 每个待在自己隔离的代码检出里，互不干扰

### 飞轮效应

```
云端例程发现 → 写入文件 → 本地循环读取 → 执行 → 大活拉起集群
                                                ↓
        每周蒸馏，更新 CLAUDE.md ← 结果回流 ←────┘
```

> "这套飞轮一转，你的智能体每周都比上周更聪明。"

### 入门级：七条可以直接抄的循环

七条 slash 命令，七个并行循环：
- 盯 PR
- 挖 Slack 反馈
- 清僵尸 PR
- 给 issue 分类
- 把反复纠正的规则沉淀进 CLAUDE.md
- ...（全为会话级，关终端即停）

---

## 五、提示词工程死了吗？没有

> "提示词没死，它只是从你手里钻进了 `/loop` 和 `/goal`。你不再一句句敲，现在写进循环、反复复用。所以你苦练的提示词非但没作废，反而更值钱了。"

**Claude Code 命令：**
- `/loop`：让一段提示词按你设的间隔反复跑
- `/goal`：让智能体一直干，直到你写的那个条件为真才停

**Codex 命令：**
- `/goal` + 定时任务（Automations）

**ChatPRD 创始人 Claire Vo 的判断：**

> "你要是在 Claude Cowork 里设过一个定时任务，那么恭喜，你已经写过一个循环了。"

---

## 六、历史脉络

| 时间 | 事件 |
|------|------|
| 2022 | ReAct 论文 |
| 2023 | AutoGPT |
| 2025.7 | Geoffrey Huntley 提出 Ralph 循环（一行 bash 反复喂同一句提示词） |
| 2025.9 | Simon Willison 写「设计智能体循环」 |
| 2026 春 | Codex 和 Claude Code 同时上了 /goal |
| **2026.6** | **Loop Engineering 概念引爆** |

---

## 七、三大挑战

### 1. 成本

- 多个智能体叠着子智能体，动态工作流吃掉预期 5-10 倍 token
- Steinberger 敢玩是因为"拥有无限 token"（OpenAI 福利），普通人没有

### 2. 失控

Anthropic 自己点名的三种坑：

| 坑 | 表现 |
|-----|------|
| **偷懒** | 50 项安全问题做了 20 项就说"搞定" |
| **自夸** | 给自己的活打高分 |
| **漂移** | 跑了很多轮后，第 47 轮悄悄忘了"别做 X" |

> 正因如此，「制造者」和「检查者」必须拆开。

### 3. 质疑

> Reddit 上不少人觉得：循环工程本质就是 ReAct、agent loop、任务调度换个名字加层包装，新瓶装旧酒。

疯转的辛普森梗图：
> "我是提示词工程师" → 钻进灌木丛 → "提示词工程根本不算什么" → "我现在是上下文工程师" → "我现在是循环工程师"

就连 Anthropic 也坦白：人均日合并代码量是 2024 年的 8 倍——"这个数几乎肯定高估了真实生产力"。

---

## 八、核心结论

### 会写循环才是你的护城河

- 模型是水电煤：谁都能接，谁接的都一样
- 真正形成复利的：你那套循环、攒下来的技能、每周蒸馏的 CLAUDE.md
- Boris 卸载 IDE 的底气：他的产出不依赖于某一次对话，而是依赖**系统**

### 三个提醒（Osmani）

1. **验证还得你自己过**：循环报"完成"，只是它的说法，不是真做对了的证明
2. **理解债越欠越多**：循环写的代码越堆越高，每个没读就合的 PR 都在加宽这道沟
3. **最险的**：同一个循环，用在你懂的活上是杠杆；用来逃避理解，就是在加速自己的下滑

---

## 参考资料

- Addy Osmani - Loop Engineering：https://addyosmani.com/blog/loop-engineering/
- Peter Steinberger 原推：https://x.com/steipete/status/2063697162748260627
- @Av1dlive 的 THE HIVE：https://x.com/Av1dlive/status/2064292484856041558
- Business Insider - What Are Loops：https://www.businessinsider.com/what-are-loops-ai-engineering-tips-2026-6
- loops.elorm.xyz（预置循环库）：https://loops.elorm.xyz/
- Simon Willison - Designing agent loops（2025.9）
- The New Stack - Loop Engineering：https://thenewstack.io/loop-engineering/
- Forbes - Loop Engineering：https://www.forbes.com/sites/lanceeliot/2026/06/17/loop-engineering-is-fully-making-the-rounds-for-boosting-generative-ai-and-agentic-ai/
