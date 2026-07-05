# ContinuityKit / Agent Continuity Protocol Specification

Version: 0.1 draft

## 1. Principle

Continuity is connection quality, not uninterrupted existence.

ACP improves agent continuity by preserving and validating structured state across turns and sessions.

ACP does not claim that an agent has subjective experience, offline awareness, or uninterrupted consciousness.

```
┌────────────────────────────────────────────────────┐
│                  Agent Runtime                      │
│                                                     │
│  Session Start                                      │
│      │                                              │
│      ▼                                              │
│  pre_llm_call ─── snapshot injection                │
│      │           heartbeat trigger                  │
│      ▼                                              │
│  User Message ─── LLM ─── Response                  │
│      │                    │                         │
│      │            [HEARTBEAT] block                 │
│      │                    │                         │
│      ▼                    ▼                         │
│  post_llm_call ─── validate blocks                  │
│      │           check boundary claims              │
│      ▼                                              │
│  Session End ─── write snapshot ─── RELAY / log     │
│                                                     │
└────────────────────────────────────────────────────┘
```

## 2. Runtime Model

```yaml
runtime_stack:
  state_machine:
    role: "calculate triggers and maintain counters"
  pre_llm_injection:
    role: "inject compact historical context and runtime rules"
  llm_generation:
    role: "generate user-facing response and required structured blocks"
  post_llm_validation:
    role: "validate structure, required fields, and boundary claims"
  append_only_log:
    role: "persist evidence and events"
```

## 3. Required Event Types

```yaml
event_types:
  - heartbeat
  - session_snapshot
  - validation_failed
  - memory_promoted
  - memory_superseded
  - boundary_rewrite
```

## 4. Heartbeat Policy

```yaml
heartbeat_policy:
  enabled_scope: "active_conversation"
  default_mode: "normal"
  modes:
    light:
      every_n_turns: 8
      max_tokens: 150
    normal:
      every_n_turns: 5
      max_tokens: 250
    deep:
      every_n_turns: 3
      max_tokens: 400
  triggers:
    every_n_turns: true
    after_topic_shift: true
    after_major_decision: true
    before_end_snapshot: true
    idle_closure:
      enabled: true
      trigger: "next_model_call_after_idle_gt_20min"
      idle_threshold_minutes: 20
      max_once_until_next_user_message: true
      cost_while_idle: 0
```

## 5. Active Heartbeat

```yaml
[CONTINUITY_HEARTBEAT]
type: active
timestamp: "YYYY-MM-DDTHH:mm:ssZ"
trigger:
  kind: "every_n_turns | topic_shift | major_decision | before_snapshot"
  reason: ""
attention_focus: ""
active_assumptions:
  - ""
unresolved_threads:
  - ""
user_mode_observed:
  description: ""
  evidence:
    - ""
next_best_move: ""
confidence:
  heartbeat_quality: "low | medium | high"
[/CONTINUITY_HEARTBEAT]
```

## 6. Closure Heartbeat

Closure heartbeat is emitted on the next model call after a meaningful idle gap.

```yaml
[CONTINUITY_HEARTBEAT]
type: closure
timestamp: "YYYY-MM-DDTHH:mm:ssZ"
trigger:
  kind: "idle_closure"
  reason: "idle gap exceeded threshold and no closure heartbeat has fired for this gap"
last_attention_focus: ""
unresolved_threads:
  - ""
pending_decisions:
  - ""
snapshot_recommendation: ""
confidence:
  heartbeat_quality: "low | medium | high"
[/CONTINUITY_HEARTBEAT]
```

## 7. Session Snapshot

```yaml
[SESSION_SNAPSHOT]
timestamp: "YYYY-MM-DDTHH:mm:ssZ"
task_anchor: ""
attention_focus:
  - ""
active_memories:
  - name: ""
    reason_used: ""
unfinished_reasoning:
  - ""
interaction_mode:
  observed: ""
  evidence:
    - ""
next_best_action:
  - ""
confidence:
  snapshot_quality: "low | medium | high"
[/SESSION_SNAPSHOT]
```

## 8. Memory Promotion

Heartbeats are ephemeral by default.

```yaml
memory_promotion_policy:
  heartbeat_default_status: "ephemeral"
  promote_only_if:
    - "same unresolved thread appears repeatedly"
    - "user explicitly confirms importance"
    - "state affects future behavior"
    - "claim has stable evidence"
  do_not_promote:
    - "temporary assumptions"
    - "single-turn speculation"
    - "uncertain user state inference"
```

## 9. Provenance

All state claims should include provenance when stored.

```yaml
provenance:
  source_type: "user_message | assistant_message | heartbeat | snapshot | tool_log"
  source_id: ""
  observed_at: ""
  confidence: "low | medium | high"
```

## 10. Boundary Rules

The agent must not claim offline thinking, waiting, memory, or subjective continuity unless supported by explicit logs.

Allowed:

```text
According to the previous snapshot, the recovered focus is...
```

Avoid:

```text
I was thinking about this while you were away.
```
