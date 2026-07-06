# ContinuityKit

> 你的 AI Agent 会漂移、会失忆、会一本正经地胡说。这个项目修的就是这个。
> Your AI agent drifts, forgets, and confabulates. This fixes that.

---

## 四个你一定踩过的坑 / Four Problems You've Definitely Hit

如果你的 AI Agent 用了超过 15 分钟：

If you've used an AI agent for more than 15 minutes:

**1. 漂移 / Drift。** Agent 抓住一个细节就再也回不来。二十轮以后你发现它在优化一个你根本没提过的东西——而它从来没停下来问过自己：「等等，我现在到底在干嘛？」

The agent latches onto a tangent and never comes back. Twenty turns later you realize it's been optimizing something you didn't ask for — and it never once stopped to ask "wait, what am I actually doing?"

**2. 失忆 / Session amnesia。** 你关掉终端。下次打开，Agent 完全不记得上次聊到哪了。「我们之前在改数据库迁移脚本。」——空白回应。你重新解释一切。

You close the terminal. Next session, the agent has no idea where you left off. "We were working on the database migration." Blank stare. You re-explain everything.

**3. 假设腐败 / Assumption rot。** 二十轮前的一个临时猜测——「可能是配置文件的问题」——变成了铁板钉钉的「事实」。Agent 在这个猜测上继续推理，从没回头验证过。三小时后，你的整条调试路径建在流沙上。

A temporary guess from 20 turns ago — "maybe the error is in the config" — becomes treated as established fact. The agent builds on it without ever revisiting it. Three hours later, your entire debugging path is built on quicksand.

**4. 幽灵连续性 / Ghost continuity。** Agent 说「我一直在监控这个」或者「我之前提到过」——但它根本没有。没有任何记录证明它做过这些事。你分不清哪些是真的做过的，哪些只是它语气听起来很自信。

The agent says "I've been monitoring this" or "as I mentioned earlier" — but it's confabulating. There's no record of any such action. You can't tell what was actually done and what the agent just *sounds confident about*.

**这些不是记忆问题。** Agent 不缺记忆。它缺的是——知道自己知道什么、自己在假设什么、自己忘了什么。这叫元认知。再好的上下文压缩也补不上这个缺口。

These are not memory problems. The agent has plenty of memory. What it lacks is **knowing what it knows, what it's assuming, and what it's forgotten**. That's metacognition — and no amount of context compression will give it that.

---

## ContinuityKit 做什么 / What ContinuityKit Does

三层。每层对号入座解决上面那些坑。

Three layers. Each targets specific problems.

### 1. 心跳 → 克漂移 / Heartbeats → Stops Drift

每约 5 轮，Agent 暂停，自己汇报当前到底在干什么：

Every ~5 turns, the agent pauses and self-reports what's actually happening:

```
[CONTINUITY_HEARTBEAT]
attention_focus: "正在排查 hook 子进程的 GBK 编码崩溃"
active_assumptions:
  - "pipeline.py 已验证输出纯 ASCII"
  - "bridge.mjs 的 asciiJson 覆盖全部 stdout"
unresolved_threads:
  - "0xB2 字节的精确来源未定位"
  - "心跳漏发了 8 轮——执行可靠性有缺口"
confidence: high
[/CONTINUITY_HEARTBEAT]
```

这不是记忆。这是在 token 预测的惯性流里**强行插一个中断**。LLM 不会「决定」自我反思——hook 强制它做。五轮没出心跳？下一轮 prompt 里直接注入提醒：「你漏了心跳。校准。」

This is not memory. It's a **forced interrupt in the token-prediction stream**. The LLM doesn't "decide" to self-reflect — the hook makes it happen. Five turns without a heartbeat? The next prompt includes a reminder: "You missed your heartbeat. Recalibrate."

### 2. 会话快照 → 克失忆 / Session Snapshots → Fixes Session Amnesia

会话结束时，Agent 写一份结构化索引——不是全部内容，只记关键的：

When a session ends, the agent writes a structured index — not everything, just what matters:

- 我在做什么？（task_anchor）/ What was I doing?
- 我在关注什么？（attention_focus）/ What was I focused on?
- 我用了什么规则？（active_memories）/ What rules was I using?
- 什么推理没收束？（unresolved_threads）/ What reasoning was NOT finished?
- 下一步该干嘛？（next_best_action）/ What should I do next?

下次会话：快照在 LLM 第一次调用前注入。Agent 不是「记住了」——它是读了自己的状态。就像开发者读自己的 commit message。不用重新解释。没有空白回应。

Next session: the snapshot is injected before the first LLM call. The agent doesn't "remember" — it reads its own state. Like a developer reading their own commit messages. No re-explaining. No blank stares.

### 3. RELAY 协议 → 克幽灵连续性 / RELAY Protocol → Ends Ghost Continuity

一个活文件追踪每一次会话：

A living file tracks every session:

- 会话指针：一行摘要，最新在上 / Session pointers: one-line summaries, newest first
- 待办任务：已承诺但未完成的 / Pending tasks: committed but not completed
- 已完成：可验证的记录——什么时间做了什么 / Completed: verifiable record — what was done, when

不再有「这个我们是不是搞完了？」。不再有「Agent 说做过了但我没法确认」。每一项完成都有时间戳，追溯到具体会话。

No more "did we already finish that?" No more "the agent says it did something but I can't confirm." Every completed item is timestamped and traceable to a specific session.

---

## 一个诚实的前提 / The Catch

心跳和快照是 **Agent 自己写的**。一个会胡说的 Agent，心跳也可能胡说。

Heartbeats and snapshots are **written by the agent**. An agent that confabulates can confabulate its heartbeat too.

ContinuityKit 不能阻止胡说——它让胡说**可被审计**。伪造的心跳看得见。嘴上说「我一直在做这个」但 RELAY 空空如也——那就是红旗。这个协议不能让 Agent 变诚实；它让不诚实无法隐藏。

ContinuityKit doesn't prevent confabulation — it makes it **auditable**. A fabricated heartbeat is visible. An empty RELAY after "I've been working on this all day" is a red flag. The protocol doesn't make agents honest; it makes dishonesty detectable.

如果需要主动防胡说，搭配输出验证器如 [Hermes Guard](https://github.com/phamhien4278n-hub/hermes-guard) 使用。

For active confabulation prevention, pair ContinuityKit with an output validator like [Hermes Guard](https://github.com/phamhien4278n-hub/hermes-guard).

---

## 实验验证 / Experimental Validation

对照实验（每组 N=4，30 轮会话）：

Controlled A/B test (N=4 per group, 30-turn sessions):

| 组 / Group | 配置 / Setup | 任务连贯性 / Task coherence |
|-------|-------|:---:|
| A | 无心跳、无快照 / No heartbeat, no snapshot | 13.50 |
| **B** | **ContinuityKit 协议** / **ContinuityKit protocol** | **19.50** |

+6.00。可测量。完整数据：`docs/experiment-20260705.md`。更大规模验证进行中。

+6.00. Measurable. Full data: `docs/experiment-20260705.md`. Larger-scale validation in progress.

---

## 跟上下文压缩的区别 / vs. Context Compression

上下文压缩（Codex、Claude Code、Hermes v0.18）解决的是另一个问题：

Context compression (Codex, Claude Code, Hermes v0.18) solves a different problem:

| | 上下文压缩 / Context Compression | **ContinuityKit** |
|---|---|---|
| **解决什么 / Problem** | 「上下文窗口满了」/"Context window is full" | 「Agent 在漂移/失忆/胡说」/"Agent is drifting/forgetting/confabulating" |
| **做什么 / What it does** | 摘要历史，省 token / Summarizes history to save tokens | 强制周期性自我校准 / Forces periodic self-calibration |
| **Agent 能看见吗 / Agent sees it?** | 不能——黑盒 / No — opaque | **能**——心跳就在输出里 / **Yes** — heartbeat is right there |
| **你能审计吗 / Can you audit it?** | 不能——内部的 / No — internal | **能**——每个快照和 RELAY 条目都是文件 / **Yes** — every snapshot is a file |
| **绑定什么 / Framework** | 绑定运行时 / Tied to runtime | **任何**支持 hook+文件读写的运行时 / **Any** runtime with hooks + file I/O |

两者互补。都用。压缩管上下文窗口不爆。ContinuityKit 管 Agent 不走偏。

They're complementary. Use both. Compression keeps the context window manageable. ContinuityKit keeps the agent honest.

---

## 适合谁 / Who This Is For

| 如果你…… / If you... | ContinuityKit 能帮你…… / ContinuityKit helps you... |
|---|---|
| 跑跨度几小时甚至几天的 Agent 任务 / Run agents on tasks spanning hours/days | 用心跳防止跑偏 / Stop drift with periodic heartbeats |
| 经常重启会话 / Frequently restart sessions | 用快照秒恢复 / Resume instantly with snapshots |
| 一个 Agent 同时跟多个项目 / Juggle multiple projects with one agent | 用 RELAY 跨会话追踪任务 / Track tasks across sessions with RELAY |
| 需要 Agent 操作的审计轨迹 / Need audit trails for agent actions | 每个声明都有来源 / Every claim has provenance |
| 做陪伴/角色类 Agent / Build companion/character agents | 有连续性但不做虚假意识声明 / Continuity without fake consciousness |
| 用任何 Agent 框架 / Use any agent framework | 框架无关——纯文本 + hook / Framework-agnostic — plain text + hooks |

---

## 不是啥 / What This Is NOT

- **不是上下文压缩。** 我们不动上下文窗口。 / Not a context compressor. We don't touch your context window.
- **不是记忆系统。** 我们管状态*怎么*创建和验证，不管*存什么*。可以和任何记忆后端搭配。 / Not a memory system. We govern *how* state is created and verified, not *what* is stored. Works alongside any memory backend.
- **不是意识框架。** 连续性 = 连接质量，不是主观体验。我们明确拒绝离线连续性声明。 / Not a consciousness framework. Continuity = connection quality, not subjective experience. We explicitly reject offline-continuity claims.
- **不是库（目前）。** v0.1 是协议规范 + schema。参考实现计划在 v0.2。 / Not a library (yet). v0.1 is a protocol + schemas. Reference implementations planned for v0.2.

---

## 快速示例 / Quick Example

一次真实调试会话中发出的心跳：

A heartbeat emitted during a real debugging session:

```text
[CONTINUITY_HEARTBEAT]
attention_focus: "排查 Hermes v0.18 hook 子进程的 GBK 编码崩溃"
active_assumptions:
  - "pipeline.py 和 bridge.mjs 已验证——两者均输出纯 ASCII"
  - "0xB2 错误是 cosmetic——daemon 线程崩，不影响主流程"
unresolved_threads:
  - "0xB2 精确来源未最终分离（大概率是 stderr 管道编码问题）"
  - "心跳执行可靠性——本会话漏了 8 轮"
user_mode_observed:
  description: "诊断模式——读文件、grep、hex dump hook 输出"
  evidence:
    - "用户提供了 Thread-400+ 编号的新错误日志"
    - "用户选择接受为 cosmetic bug 而非修改 hook 配置"
confidence: high
[/CONTINUITY_HEARTBEAT]
```

内容遵循 [`schemas/heartbeat.schema.yaml`](schemas/heartbeat.schema.yaml)。

Content follows [`schemas/heartbeat.schema.yaml`](schemas/heartbeat.schema.yaml).

---

## 仓库内容 / Repository Contents

```text
docs/spec.md                     — 完整协议规范 / Full protocol specification
schemas/                         — 心跳、快照、RELAY 指针、状态 / heartbeat, snapshot, RELAY pointer, state
examples/                        — pre_llm 注入、post_llm 验证 / pre_llm injection, post_llm validation
```

---

## 快速上手 / Getting Started

1. 读协议规范：[`docs/spec.md`](docs/spec.md) / Read the spec
2. 设好 `pre_llm_call` hook（模板：`examples/pre_llm_injection.yaml`）/ Set up a pre_llm hook
3. 按 `schemas/relay_session_pointer.schema.yaml` 初始化 RELAY 文件 / Initialize a RELAY file
4. 在 Agent 的 system prompt 中加入心跳发射指令 / Add heartbeat emission to your agent's system prompt
5. （推荐）加 `post_llm_call` 验证（`examples/post_llm_validation.yaml`）/ (Recommended) Add post_llm validation

Hermes、Claude Code、Codex 的参考实现计划在 **v0.2**。

Reference implementations for Hermes, Claude Code, and Codex planned for **v0.2**.

---

## 存储后端 / Storage Backends

ContinuityKit 定义**存什么**状态。你选**存哪里**。

ContinuityKit defines **what** to store. You choose **where**.

| 后端 / Backend | 最适合 / Best for | 复杂度 / Complexity |
|---------|----------|:--:|
| 纯文件 / Flat file (YAML/JSON) | 原型开发、单 Agent / Prototyping, single-agent | 低 / Low |
| SQLite | 生产环境、审计追溯 / Production, audit trails | 中 / Medium |
| [Hindsight](https://github.com/nousresearch/hindsight) | 多 Agent、语义搜索 / Multi-agent, semantic search | 高 / High |

---

## 许可证 / License

源码可得，PolyForm Noncommercial License 1.0.0。商业使用需购买许可。详见 [LICENSE.md](LICENSE.md) 和 [COMMERCIAL_LICENSE.md](COMMERCIAL_LICENSE.md)。

Source-available under PolyForm Noncommercial License 1.0.0. Commercial use requires a paid license. See [LICENSE.md](LICENSE.md) and [COMMERCIAL_LICENSE.md](COMMERCIAL_LICENSE.md).
