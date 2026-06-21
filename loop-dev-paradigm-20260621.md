# Loop 开发范式 —— Oracle-First Loop Development

> **类型**：方法论 / 实践范式（非行业竞品报告）
> **报告日期**：2026-06-21 · **资料截止**：2026-06（社区高互动帖）
> **基础**：Addy Osmani《Loop Engineering》（见 [`loop-engineering-20260621.md`](./loop-engineering-20260621.md)）+ Claude Code 社区实践总结 + elorm `loops.elorm.xyz`（prior art）
> **首个 worked example**：excore（DEX 撮合引擎，独立仓库）

---

## 0. 一句话

> **Loop 的价值上限 = 它那个「可机器判定、且 loop 改不动」的预言机（oracle）的强度。** 机械全是内置商品件；真正的工程 = 先备好预言机 + 一层薄薄的项目配置。

这是对主流叙事（Boris Cherny「My job is to write loops」/ 从 prompt engineering 转向 loop engineering）的**补一刀**：社区把 verification 当「仍在你身上」的注意事项；本范式把它**提成前置条件**——没有预言机就没有可信 loop，先建预言机，再谈循环。

---

## 1. 背景：从 prompt 到 loop

过去两年的用法是「你写 prompt、读结果、再写下一个」——你全程握着工具。loop engineering 是**把你从"提示 agent 的人"这个位置上换下来**：你设计一个系统去发现工作、派发、验证、记录、决定下一步，让系统去戳 agent，而不是你。机械件如今直接 ship 在产品里，不再需要自己写一堆 bash 维护。

**所以工程量不在机械，在两件事**：① 建/接预言机；② 一层薄薄的项目配置。下面把两者讲清。

---

## 2. 三命令分工（机械层，全内置）

| 命令 | 维度 | 作用 | 治什么 |
|---|---|---|---|
| `/goal` | **深度** loop-until-done | 评估现状→执行→检查 rubric→重复，直到达成或触限 | 专治 *agentic laziness*（提前宣布完成） |
| `/loop` | **持续** cadence | prompt 按间隔重跑（固定或自定步），夜跑 / 监控；配 schedule 实现 cron | recurring 任务 |
| **动态 Workflow** | **宽度** scale | 动态生成编排脚本、并行 fan-out 子代理、分阶段、双验证；可保存复用 | 大规模迁移 / 审计 / 并行上百项 |

**组合式**（社区典型）：
- `/loop 10m /goal: <rubric>` = 心跳 + 深度。
- Workflow 包宽度，其内部子代理再用 `/goal` 抠深度。

典型 prompt 结构：
```
/loop 10m /goal: [清晰目标 + 可检查 rubric + 验证方式 + 记忆更新规则]
```
或：「Use a workflow to [任务]，严格按阶段执行并验证。」

机械还包括：`isolation:worktree`（并行隔离）、hooks（生命周期挂钩）。**这一层 0 工程，纯组合。**

---

## 3. 核心论点：Oracle-First

社区的三大共识——「checkable rubric」「builder/verifier 分离」「explicit stop condition」——**全是「有预言机」的下游战术**：rubric 就是预言机写成可检查标准；verifier 就是去**跑**预言机；stop condition 就是**预言机绿**。所以预言机是统一原理，其余是它的战术。

**预言机的两个硬要求**：
1. **可机器判定**：一条命令给出 pass/fail（exit code / 状态串），不是模型自评。
2. **loop 改不动**：loop 不能通过弱化测试/改 rubric 来"变绿"——否则就是给自己作弊。

### 预言机就绪度三档（最可迁移的判据）

| 档 | 特征 | 上 loop 的姿势 |
|---|---|---|
| **Oracle-native** | 测试/类型/规格本就强（编译器、协议、excore） | 现在就能上 |
| **Oracle-buildable** | 能补金样/属性/快照测试 | **先建预言机，再 loop**（这一步本身就是 TDD 教的） |
| **Oracle-weak** | UI/审美/开放研究，无客观判据 | loop 收益低，人贴得更紧，缩短循环 |

> 给任何新项目上 loop 前，先问：**我的预言机在哪档？** 不在 native，先把它搬上去。

---

## 4. 分层模型：机械（商品件） vs 自著层（真工程）

机械见 §2。**自著层 = 每个项目重填一次的 3 件 + backlog**：

### 件 1：语义守则（预言机只读 + builder/verifier 契约）

写进 maker/checker 的契约（**语义约束，不做物理拦截**——现代模型足够守得住，硬拦反而误伤无关改动）：

> - Builder 禁止修改测试 / 规格 / rubric 来强制变绿。借 elorm 原话：*"Do not modify the check command or exit criteria to force success."*
> - Verifier 只信命令输出，不信实现者的自我陈述。elorm 原话：*"Trust only command output, not prior claims."*
> - Verifier 对抗审查时额外检查一条：实现是否动了预言机文件。

可选**零摩擦补强**（纯可见、不拦）：verifier 跑 `git diff --stat <预言机目录>`，非空就在状态里标给人工门看。

### 件 2：memory / state 脊柱

loop 跨会话会忘光，记忆必须在磁盘、不在 context。一个会话外的状态文件（`PROGRESS.md` / `state.md` / Linear board）持有：当前阶段、backlog、已完成（带 commit SHA）、当前红、下一步决策。
教训写入协议（社区）：**Fail → Investigate → Verify → Distill → Consult**。

### 件 3：项目知识 skills

把项目约定/构建步骤/踩过的坑写成 skill，让每次迭代**复利**而非从零重推（intent debt）。配 CLAUDE.md house rules——错误率数据（Karpathy + 社区反复验证）：

| CLAUDE.md 状态 | 错误率 |
|---|---|
| 无 | ~41% |
| 4 条基础规则 | ~11% |
| 12 条完善规则 | ~3% |

> 注意：是**条目要全、每条一行可检查**，不是写长 prose。「精简」精简的是表述，不是规则数。

### backlog（路线图 → 可验证增量）

loop 不是开放式发现，是沿既定路线图挑「下一个朝当前门禁推进的增量」。架构由人定，loop 填实现。

---

## 5. 执行结构：检查点 + 人工门

分阶段、每大步 checkpoint，绝不在坏基础上叠加：

```
Plan(提案) → Approve(人工门) → Build(maker) → Verify(独立 verifier) → Save(绿则提交+记状态) → Next
```

**人工门（stay the engineer）**——loop 只能提议+停，不能自决：
1. 任何**动预言机**的提议（改了 ground truth，verifier 就失真）。
2. **架构 / 阶段跃迁**（架构是人的判断）。
3. **合并**（PR review）。

文章的三条警告对应这里：verification 仍在你（"done" 是 claim 不是 proof）、comprehension debt（不读 loop 写的码，理解会烂）、cognitive surrender（别因为 loop 能跑就停止有观点）。

---

## 6. 成本纪律

- **模型分层**：只读 / 机械阶段用便宜模型；关键验证 / 架构用强模型。Workflow 可按 stage 设 `model` / `effort`。
- **显式停止条件 + max iterations**：防 agentic laziness 的反面——无限循环烧 token。
- 并行 Workflow 上百子代理全用贵模型，几分钟能干掉额度——宽度任务尤其要分层。

---

## 7. 填空模板（任何项目复制即用）

```
项目：________
[1] 预言机：命令 = ________  通过判据 = ________  就绪度档 = native/buildable/weak
    (buildable→先补测试再继续)
[2] 语义守则：禁改预言机 + verifier 只信命令输出   (写进 maker/checker 契约)
[3] state 脊柱：________.md（阶段 / backlog / 已完成 / 红 / 下一步）
[4] skills：________（项目知识，让迭代复利）+ CLAUDE.md house rules
[5] backlog：路线图拆成可验证增量 ________
[6] 模式选择：深度 /goal · 持续 /loop · 宽度 Workflow
[7] 人工门：动预言机 / 架构跃迁 / 合并
[8] 成本：只读便宜模型 + 关键验证强模型 + max iterations
```

---

## 8. Worked Example：excore

excore 的开发哲学（测试先行、预言机是 ground truth、断言只增不松）**本身就等于本范式的前置条件**——它按构造满足 oracle-first（先写 spec+vectors+INV 才写实现）。这是用它当蓝本的原因：把"最难的预言机"已经做完，注意力留给范式结构。

| 自著件 | excore 实例 |
|---|---|
| 预言机 | `vectors/`（金样向量）+ `invariants`（INV-1..7）+ `reference/`（参考实现）；判据 = `cargo test` 绿 + `clippy` 零告警 +（框架阶段）C 差分逐帧状态根 == 参考实现 |
| 语义守则 | maker 禁改 `spec/`/`vectors/`/`invariants`；checker 对抗审 spec |
| state 脊柱 | `.loop/state.md` + 记忆目录 |
| skills | `excore-verify`（跑全套门禁）、`excore-vector-author`（手算金样向量） |
| backlog | Stage 2a（Oracle trait 接缝 → 双线程 §7.8 → C 差分）→ 2b → 高性能 → 高可用 |
| 专用 loop 库 | build=实现增量 · harden=B 属性 + D 类分布式 · test=补向量 · refactor |

---

## 9. 踩坑（社区血泪）

| 坑 | 后果 | 解 |
|---|---|---|
| 目标模糊（"改进代码库"） | drift（发明没人要的功能）/ 提前完成 / 无限烧钱 | checkable rubric |
| 无项目结构 / CLAUDE.md | 一直猜上下文，错误率暴增 | house rules（见 §4 数据） |
| 无 verifier | 自信满满但输出错的（最隐蔽） | 独立 verifier，只信命令输出 |
| 无 checkpoint / memory | 错误层层叠加，几小时后才发现基础烂了 | §5 检查点 + §4 state 脊柱 |
| 成本失控 | 并行贵模型几分钟干掉额度 | §6 模型分层 + max iterations |
| 把 AI 当聊天机器人 | 一直手动 prompt，不设计系统 | 写 loop |

---

## 10. 实操精炼（经 excore 验证，2026-06）

把上文范式落到一个真实 Claude Code 仓库（excore）后，沉淀出 5 条更贴实操的硬约束：

1. **AGENTS.md 为人类拥有的单一约束源**（`CLAUDE.md` 软链指向它）。跨工具（Claude Code / Codex 都读）、loop 只读。架构约束 + 不变量都进它，不再散落到独立 spec 文档。
2. **oracle = AGENTS.md（约束/不变量）+ vectors（手算金样数据）**，且只有这两样 loop 只读；其余（实现代码、语义 doc）皆 loop 可改。边界从"spec 全家"收窄到"约束 + 数据"两点，更干净、更难被绕过。
3. **代码为唯一事实来源**（单人/小队尤其）：散文 spec 会与代码 drift。把语义 math 收进**实现它的代码的模块 doc**（零 drift），不变量收进 AGENTS.md，向量 schema = 解析它的 serde 结构 → 删掉独立 spec 目录。**唯一不可收的是 vectors**——它是独立于实现的手算裁判，删了 loop 就能自定义真相。
4. **每个 repo 的 loop 实例自包含**：`.loop/state.md` 不引用外部范式文档（本文档）。范式是"怎么做"的知识（可共享），实例是"这个 repo 的状态"（自包含），解耦——否则跨仓库依赖会在 clone / 迁移时断裂。
5. **oracle 按版本目录**（`vN/{reference, vectors}`），实现追着 oracle 版本走。"版本"是业务契约的版本（v1 简单基线 → v2 生产业务 → v3 衍生品），不是代码版本；先用简单 oracle 验证架构/性能，再迁生产业务。

> 净效果：一个仓库的"人类拥有层"压到两处——`AGENTS.md`（约束 + INV）+ `vN/vectors`（金样）。loop 在其外自由迭代，碰这两样必过人工门。

## 11. 参考 / Prior Art

- **Addy Osmani《Loop Engineering》** —— 五支柱（automations / worktrees / skills / plugins-connectors / sub-agents）+ memory；全文存档 [`loop-engineering-20260621.md`](./loop-engineering-20260621.md)。
- **Boris Cherny（Claude Code 负责人）/ Peter Steinberger** —— "My job is to write loops" / "design loops that prompt your agents"。
- **elorm `loops.elorm.xyz`** —— 社区起步模板目录（Ship-PR-Until-Green / Independent-Verifier-Pass / CI-Failure-Watcher / Deploy-Verification / Pre-Commit & Post-Edit Guard）。**定位：prior art，非权威**——骨架浅（每个 ~5-15 行 prompt）、JS/Node 中心、捐赠制无仓库可审计、缺 memory/人工门/成本分层。**取法**：借命名与金句（见 §4），自行复刻骨架并补齐缺失支柱；要依赖就本地快照。

---

## 12. 总结

Prompt engineering 不是过时，而是**杠杆点上移**了：从"写好一次提示"到"设计可见、可验证、可停止的系统"。本范式的主张是——**这套系统的地基是预言机**。机械是商品件（`/goal`/`/loop`/Workflow/worktree），照搬即可；工程在于先把预言机搬到 native 档，再叠那 3 件薄薄的自著层 + 路线图 backlog。excore 因为按构造就是 oracle-first，成为检验这套范式的第一个活体。

> Build the loop. But build it like someone who intends to stay the engineer, not just the person who presses go. —— Addy Osmani
