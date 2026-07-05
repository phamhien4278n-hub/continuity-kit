# ContinuityKit — Give Your AI Agent a Heartbeat

> **心跳系统 · Heartbeat System · Agent Continuity Protocol · 会话指针 · Session Relay**
> 让你的 AI Agent 拥有心跳。记忆不够，让 Agent 活起来。
> Memory is not enough. Give your agent a heartbeat.

**ContinuityKit** is a source-available toolkit and protocol for building AI agents that can pause, resume, self-check, and remember — without turning temporary thoughts into permanent truth. Think of it as a **heartbeat for any AI agent**: periodic state checkpoints that make long-running agents feel connected and auditable instead of starting from scratch every session. Framework-agnostic — works with Hermes, Claude Code, Codex, or any runtime that supports hooks.

It gives your agent a practical continuity layer:

- **Heartbeats** — in-session checkpoints capturing what the agent is focused on right now.
- **Session snapshots** — end-of-session state indexes for high-fidelity resume after interruption.
- **Memory promotion rules** — prevent temporary assumptions from polluting long-term memory.
- **Provenance & evidence chains** — every state claim backed by a source, not a guess.
- **Runtime hooks** — `pre_llm_call` injection + `post_llm_call` validation that remind the model what to do.
- **Validators** — catch missing checkpoints, malformed state, and unsupported offline-continuity claims.

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

## What It Provides

ContinuityKit provides:

- In-session heartbeats for periodic state calibration.
- End-of-session snapshots for high-fidelity resume.
- **RELAY protocol** for cross-session retrieval — pointers, snapshots, task tracking.
- Append-only event logs for auditability.
- Memory promotion rules to prevent temporary state from polluting long-term memory.
- Runtime injection templates for `pre_llm_call` hooks.
- Validation rules for `post_llm_call` hooks.
- Evidence and provenance requirements for agent state claims.

## Why It Matters

Most AI agents are stateless or rely on loosely managed memory. This causes:

- Lost context after interruptions.
- Unclear boundaries between current state and historical summaries.
- Memory pollution from temporary assumptions.
- Unverifiable claims such as "I was thinking while you were away."
- Reliance on model self-discipline instead of external state management.

ContinuityKit treats continuity as an engineering property:

```yaml
continuity:
  definition: "connection quality, not uninterrupted existence"
  goals:
    - reliable resume
    - auditable state
    - evidence-based memory
    - lower reliance on model self-discipline
```

## Who This Is For

- **Agent framework builders** who need auditable state management across turns and sessions.
- **AI application developers** dealing with multi-turn tasks, interrupted workflows, or long-running agents.
- **Companion app creators** who want continuity without making false claims about consciousness.

## What This Is NOT

- **NOT a consciousness framework** — continuity means connection quality, not subjective experience.
- **NOT a drop-in library** — v0.1 is a protocol specification. Reference implementations are planned for v0.2.
- **NOT a replacement for memory systems** — it governs *how* memory is created and validated, not *what* is stored.

## Core Concepts

### Heartbeat

A heartbeat is a short structured checkpoint emitted during an active conversation or after a resume-worthy idle gap.

Heartbeats are ephemeral by default. They are runtime state, not long-term memory.

### Session Snapshot

A session snapshot is a compact end-of-session state index. It records what the agent was doing, focusing on, assuming, and leaving unresolved.

### Memory Promotion

Temporary state is not automatically promoted to long-term memory. Promotion requires repeated relevance, explicit user confirmation, or strong behavioral evidence.

### Runtime Hooks

ContinuityKit / ACP is designed to work with hook-based agent runtimes:

- `pre_llm_call`: inject latest snapshot, runtime rules, and heartbeat trigger.
- `post_llm_call`: validate required blocks, fields, and boundary violations.

## Repository Contents

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

## Getting Started

1. Read [`docs/spec.md`](docs/spec.md) for the full protocol specification.
2. Copy the schemas from `schemas/` into your agent runtime configuration.
3. Implement the `pre_llm_call` hook using [`examples/pre_llm_injection.yaml`](examples/pre_llm_injection.yaml) as a template.
4. Implement the `post_llm_call` hook using [`examples/post_llm_validation.yaml`](examples/post_llm_validation.yaml) for validation rules.
5. Initialize a RELAY file using [`schemas/relay_session_pointer.schema.yaml`](schemas/relay_session_pointer.schema.yaml) as the schema.
6. Use the examples in `examples/` as integration guides for different agent types.

Reference implementations for popular agent runtimes (Hermes, Claude Code, Codex) are planned for **v0.2**.

## Quick Example

A heartbeat is emitted inline during a conversation using a structured block:

```text
[CONTINUITY_HEARTBEAT]
type: active
timestamp: "2026-07-05T10:40:00+08:00"
trigger:
  kind: "every_n_turns"
  reason: "turns_since_heartbeat >= 5"
attention_focus: "Designing an auditable continuity protocol for agents"
active_assumptions:
  - "The user wants source-available public materials"
unresolved_threads:
  - "Implementation target runtime is not selected yet"
user_mode_observed:
  description: "The user is moving from concept design to publication planning"
  evidence:
    - "The user asked for public materials and a commercial-use license"
next_best_move: "Prepare a sanitized repository package"
confidence:
  heartbeat_quality: "high"
[/CONTINUITY_HEARTBEAT]
```

The `[CONTINUITY_HEARTBEAT]` / `[/CONTINUITY_HEARTBEAT]` markers are the wire format; the content inside follows the YAML schema defined in [`schemas/heartbeat.schema.yaml`](schemas/heartbeat.schema.yaml).

## License

This project is source-available under the PolyForm Noncommercial License 1.0.0.

Commercial use requires a separate paid commercial license. See [LICENSE.md](LICENSE.md) and [COMMERCIAL_LICENSE.md](COMMERCIAL_LICENSE.md).

This is not an OSI-approved open source license because commercial use is restricted.

