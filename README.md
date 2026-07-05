# ContinuityKit — Give Your AI Agent a Heartbeat

> **心跳系统 · Heartbeat System · Agent Continuity Protocol · 会话指针 · Session Relay**
> 让你的 AI Agent 拥有心跳。记忆不够，让 Agent 活起来。
> Memory is not enough. Give your agent a heartbeat.

**ContinuityKit** is a source-available toolkit and protocol for building AI agents that can pause, resume, self-check, and remember — without turning temporary thoughts into permanent truth. Think of it as a **heartbeat for any AI agent**: periodic state checkpoints that make long-running agents feel connected and auditable instead of starting from scratch every session. Framework-agnostic — works with Hermes, Claude Code, Codex, or any runtime that supports hooks.

It gives your agent a practical continuity layer / 为你的 Agent 提供一套实用的连续性层：

- **心跳 Heartbeats** — 会话内周期性状态校准，捕获 Agent 此刻的注意力焦点。 / in-session checkpoints capturing what the agent is focused on right now.
- **会话快照 Session snapshots** — 会话结束时的高保真状态索引，中断后精准恢复。 / end-of-session state indexes for high-fidelity resume after interruption.
- **记忆防污染 Memory promotion rules** — 防止临时假设污染长期记忆，晋升须有证据。 / prevent temporary assumptions from polluting long-term memory.
- **证据链 Provenance & evidence chains** — 每个状态声明都有据可查，不是猜的。 / every state claim backed by a source, not a guess.
- **运行时钩子 Runtime hooks** — `pre_llm_call` 注入提醒 + `post_llm_call` 验证输出。 / injection + validation that remind the model what to do.
- **校验器 Validators** — 捕获缺失的心跳、格式错误的状态、不实的离线连续性声明。 / catch missing checkpoints, malformed state, and unsupported offline-continuity claims.

## 简介

**ContinuityKit 是一套面向长运行 AI Agent 的连续性协议与工具包。**

它解决的不是"让 AI 假装从不断线"，而是一个更实际、更商业化的问题：

> 当 Agent 被打断、跨会话恢复、处理长期任务或管理记忆时，如何让它可靠、可审计、可治理地接上之前的状态？

把它理解为 **给你的 Agent 装上心跳**——周期性状态校准 + 会话快照 + 记忆防污染，让 Agent 不再每个新会话都像"失忆了一样从头开始"。无论你用的是 Hermes、Claude Code 还是任何支持 hook 的 Agent 运行时。

## Introduction

**ContinuityKit is a continuity protocol and toolkit for long-running AI agents.**

It does not try to prove that an agent has uninterrupted consciousness. It solves a more useful engineering problem:

> How can an agent pause, resume, checkpoint, validate, and govern memory across turns and sessions?

ContinuityKit provides heartbeats, session snapshots, memory promotion, provenance, runtime injection, and output validation so agent continuity becomes inspectable infrastructure instead of prompt folklore.

> **ContinuityKit** is the toolkit. **ACP** (Agent Continuity Protocol) is the protocol it implements — the heartbeat format, snapshot schema, and validation rules defined in [`docs/spec.md`](docs/spec.md).

## 提供什么 / What It Provides

ContinuityKit provides / ContinuityKit 提供：

- 会话内心跳，周期性状态校准。 / In-session heartbeats for periodic state calibration.
- 会话结束快照，中断后高保真恢复。 / End-of-session snapshots for high-fidelity resume.
- **RELAY 协议**，跨会话检索——指针、快照、任务追踪。 / **RELAY protocol** for cross-session retrieval — pointers, snapshots, task tracking.
- 追加式事件日志，全程可审计。 / Append-only event logs for auditability.
- 记忆晋升规则，防临时状态污染长期记忆。 / Memory promotion rules to prevent temporary state from polluting long-term memory.
- `pre_llm_call` 注入模板。 / Runtime injection templates for `pre_llm_call` hooks.
- `post_llm_call` 验证规则。 / Validation rules for `post_llm_call` hooks.
- 证据与溯源要求，Agent 状态声明须有据。 / Evidence and provenance requirements for agent state claims.

## 为什么重要 / Why It Matters

大多数 AI Agent 是无状态的，或依赖松散管理的记忆。这导致： / Most AI agents are stateless or rely on loosely managed memory. This causes:

- 中断后上下文丢失。 / Lost context after interruptions.
- 当前状态与历史摘要边界模糊。 / Unclear boundaries between current state and historical summaries.
- 临时假设污染长期记忆。 / Memory pollution from temporary assumptions.
- 无法验证的声明，如"你不在的时候我也在想你"。 / Unverifiable claims such as "I was thinking while you were away."
- 依赖模型自律而非外部状态管理。 / Reliance on model self-discipline instead of external state management.

ContinuityKit 把连续性当作工程属性来对待 / treats continuity as an engineering property:

```yaml
continuity:
  definition: "connection quality, not uninterrupted existence"
  goals:
    - reliable resume
    - auditable state
    - evidence-based memory
    - lower reliance on model self-discipline
```

## 适合谁 / Who This Is For

- **Agent 框架开发者**——需要在多轮、跨会话中管理可审计状态的。 / who need auditable state management across turns and sessions.
- **AI 应用开发者**——处理多轮任务、中断工作流或长期运行的 Agent。 / dealing with multi-turn tasks, interrupted workflows, or long-running agents.
- **陪伴类 App 创作者**——想要连续性但不做虚假意识声明。 / who want continuity without making false claims about consciousness.

## 不是啥 / What This Is NOT

- **不是意识框架**——连续性 = 连接质量，而非主观体验。 / continuity means connection quality, not subjective experience.
- **不是开箱即用的库**——v0.1 是协议规范。参考实现计划在 v0.2。 / v0.1 is a protocol specification. Reference implementations are planned for v0.2.
- **不是记忆系统的替代品**——它管*怎么存*、*怎么验证*，不管*存什么*。 / it governs *how* memory is created and validated, not *what* is stored.
- **不绑定特定存储后端**——你自带（文件、SQLite、Hindsight 等）。 / you bring your own (flat file, SQLite, Hindsight, etc.).

## 核心概念 / Core Concepts

### 心跳 Heartbeat

会话进行中或空闲回归时发出的短结构化检查点。 / A short structured checkpoint emitted during an active conversation or after a resume-worthy idle gap.

心跳默认是临时的——它是运行时状态，不是长期记忆。 / Heartbeats are ephemeral by default. They are runtime state, not long-term memory.

### 会话快照 Session Snapshot

会话结束时的精简状态索引。记录 Agent 在做什么、关注什么、假设了什么、留下了什么未解决。 / A compact end-of-session state index. It records what the agent was doing, focusing on, assuming, and leaving unresolved.

### 记忆晋升 Memory Promotion

临时状态不会自动成为长期记忆。晋升需要：反复出现、用户明确确认、或强有力的行为证据。 / Temporary state is not automatically promoted to long-term memory. Promotion requires repeated relevance, explicit user confirmation, or strong behavioral evidence.

### 运行时钩子 Runtime Hooks

ContinuityKit / ACP 设计为与基于 hook 的 Agent 运行时配合使用： / designed to work with hook-based agent runtimes:

- `pre_llm_call`：注入最新快照、运行时规则、心跳触发器。 / inject latest snapshot, runtime rules, and heartbeat trigger.
- `post_llm_call`：验证必要的块、字段和边界声明。 / validate required blocks, fields, and boundary violations.

## 仓库内容 / Repository Contents

```text
docs/
  spec.md
schemas/
  continuity_state.schema.yaml
  heartbeat.schema.yaml
  session_snapshot.schema.yaml
  relay_session_pointer.schema.yaml
examples/
  pre_llm_injection.yaml
  post_llm_validation.yaml
  codex-lite.md
  companion-agent.md
LICENSE.md
COMMERCIAL_LICENSE.md
ROADMAP.md
PRIVACY.md
```

## 存储后端 / Storage Backends

ContinuityKit 定义**存什么**状态（心跳、快照、RELAY 指针），你选**存哪里**。协议本身存储无关——选适合你技术栈的后端。 / defines **what** state to store. You choose **where** to store it.

| 后端 Backend | 适合 Best for | 复杂度 | 说明 Notes |
|---------|----------|:--:|-------|
| **纯文件 Flat file** (YAML/JSON) | 原型开发、单机 Agent / Prototyping | 低 | 心跳块追加到文本文件即可，零依赖。 |
| **SQLite** | 生产环境、审计追溯 / Production | 中 | 结构化查询、事务写入、利用时间戳实现 TTL。 |
| **[Hindsight](https://github.com/nousresearch/hindsight)** | 多 Agent、语义搜索、知识图谱 | 高 | 实体链接、溯源追踪、向量搜索。适合需要查"为什么状态变了"而不仅是"变了什么"的场景。 |

### 什么时候从纯文件升级到数据库 / When to upgrade

- 需要跨会话查询快照（比如"展示上周所有未解决的线索"）。 / query snapshots across sessions.
- 需要事务保障，崩溃中写不坏状态。 / transactional guarantees.
- 多个 Agent 共享一个连续性存储。 / multiple agents sharing a continuity store.

### Hindsight 高级记忆集成 / Hindsight for advanced memory

如果你已经在用 [Hindsight](https://github.com/nousresearch/hindsight) 做 Agent 的长期记忆系统，ContinuityKit 自然叠在上面：心跳以 `unit_type: continuity_heartbeat` 入库、快照以 `unit_type: session_snapshot` 入库、RELAY 任务板映射到 Hindsight 实体图谱。开箱即得语义检索（"找到我讨论过 X 的所有会话"）和跨 Agent 连续性。 / If you already use Hindsight, ContinuityKit fits naturally on top: heartbeats become Hindsight memories, snapshots become session_snapshot units, and the RELAY task board maps to Hindsight's entity graph.

> Reference implementations for flat-file and SQLite backends are planned for **v0.2**. For now, use the schemas in `schemas/` as your storage schema and wire up the read/write calls in your hook scripts.

## 快速上手 / Getting Started

1. 阅读完整协议规范：[`docs/spec.md`](docs/spec.md) / Read the full protocol spec.
2. 把 `schemas/` 下的 schema 文件复制到你的 Agent 运行时配置中。 / Copy schemas into your agent runtime config.
3. 参考 [`examples/pre_llm_injection.yaml`](examples/pre_llm_injection.yaml) 实现 `pre_llm_call` hook。 / Implement the pre_llm hook.
4. 参考 [`examples/post_llm_validation.yaml`](examples/post_llm_validation.yaml) 实现 `post_llm_call` hook。 / Implement the post_llm hook.
5. 根据 [`schemas/relay_session_pointer.schema.yaml`](schemas/relay_session_pointer.schema.yaml) 初始化 RELAY 文件。 / Initialize a RELAY file.
6. 参考 `examples/` 中的示例作为不同 Agent 类型的集成指南。 / Use examples as integration guides.

主流 Agent 运行时（Hermes、Claude Code、Codex）的参考实现计划在 **v0.2**。 / Reference implementations planned for v0.2.

## 快速示例 / Quick Example

心跳在对话中以内联结构块发出： / A heartbeat is emitted inline using a structured block:

```text
[CONTINUITY_HEARTBEAT]
type: active
timestamp: "2026-07-05T10:40:00+08:00"
trigger:
  kind: "every_n_turns"        # 每 N 轮触发 / every N turns
  reason: "turns_since_heartbeat >= 5"
attention_focus: "正在设计可审计的 Agent 连续性协议"
active_assumptions:
  - "用户需要源码可得的公开材料"
unresolved_threads:
  - "尚未选定目标运行时平台"
user_mode_observed:
  description: "用户正在从概念设计转向发布规划"
  evidence:
    - "用户要求公开材料和商业使用许可"
next_best_move: "准备脱敏后的仓库打包"
confidence:
  heartbeat_quality: "high"
[/CONTINUITY_HEARTBEAT]
```

`[CONTINUITY_HEARTBEAT]` / `[/CONTINUITY_HEARTBEAT]` 标记是传输格式；内部内容遵循 [`schemas/heartbeat.schema.yaml`](schemas/heartbeat.schema.yaml) 中的 YAML schema。 / The markers are the wire format; the content follows the YAML schema.

## 许可证 / License

本项目在 PolyForm Noncommercial License 1.0.0 下源码可得。 / This project is source-available under the PolyForm Noncommercial License 1.0.0.

商业使用需单独购买商业许可。详见 [LICENSE.md](LICENSE.md) 和 [COMMERCIAL_LICENSE.md](COMMERCIAL_LICENSE.md)。 / Commercial use requires a separate paid commercial license.

这不是 OSI 认可的开源许可证，因为商业使用受到限制。 / This is not an OSI-approved open source license.

