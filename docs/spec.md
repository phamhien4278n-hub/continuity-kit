# ContinuityKit / Agent Continuity Protocol Specification

Version: 0.1 draft (last updated: 2026-07-05)

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
│  read RELAY ──── load snapshot ──── pre_llm_call    │
│      │           (via RELAY)      snapshot injection│
│      │                              heartbeat trigger│
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
  - relay_pointer_appended
  - relay_startup
  - relay_task_expired
  - relay_fallback_scan
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

## 11. RELAY Protocol

RELAY is the cross-session retrieval layer. Heartbeats and snapshots define *what* state to store; RELAY defines *how to find it* when a new session starts.

RELAY is designed for single-tenant use: one agent, one RELAY file. Multi-agent or multi-project isolation is not addressed in v0.1.

Without RELAY, snapshots are written but the agent has no standard way to locate the most recent one — forcing full-text scanning or reliance on model self-discipline.

```yaml
relay_responsibilities:
  session_pointers:
    role: "Lightweight index of recent sessions for fast lookup"
    stores: "session_id, timestamp, summary, keywords, snapshot_location"
  snapshot_retrieval:
    role: "Point to the N most recent full snapshots for recovery"
  task_tracking:
    role: "Carry pending tasks across session boundaries"
  cleanup:
    role: "TTL-based pruning to prevent unbounded growth"
```

### 11.1 Session Startup Protocol

Every session start should execute the RELAY startup sequence:

1. Read the RELAY file (or equivalent state store)
2. Locate the most recent session pointer
3. Load the associated session snapshot
4. Recover the last `attention_focus`, `unfinished_reasoning`, and `active_memories`
5. Present a concise summary: where the last session left off and what tasks are pending

```yaml
startup_sequence:
  step_1: "read_relay_file"
  step_2: "find_latest_session_pointer"
  step_3: "load_snapshot_from_pointer"
  step_4: "recover_state (attention_focus + unfinished_reasoning + active_memories)"
  step_5: "announce_summary (last session + pending tasks)"
  fallback: >
    If RELAY is unavailable or empty, fall back to scanning the last
    N session logs for the most recent user-assistant exchange.
```

### 11.2 Session End Protocol

When a task completes or a session ends, the agent should:

1. Determine if lessons were learned (hindsight)
2. Write an end-of-session snapshot
3. Append a session pointer to RELAY
4. Prune completed tasks and expired entries

```yaml
end_sequence:
  step_1: "collect_hindsight (lessons, decisions, new rules)"
  step_2: "write_session_snapshot"
  step_3: "append_relay_pointer (session_id, summary, keywords, snapshot_location)"
  step_4: "update_task_board (mark completed, add new pending)"
  step_5: "prune_expired (TTL-based cleanup)"
```

### 11.3 Task TTL

Tasks carried across sessions have a default TTL to prevent stale items from accumulating indefinitely.

```yaml
task_ttl_policy:
  default: "PT24H"
  description: >
    Tasks older than 24 hours without activity are automatically
    marked expired on next RELAY access.
  override: "Per-task TTL can be set explicitly"
  grace_period: >
    Expired tasks are kept for 48 additional hours in an 'expired'
    state before hard deletion, allowing manual recovery.
```

### 11.4 Pointer Limit

Session pointers grow with usage. An upper bound prevents unbounded file growth.

```yaml
pointer_limit:
  max_session_pointers: 20
  eviction: "oldest-first when limit exceeded"
  snapshot_retention: >
    Snapshots referenced by evicted pointers are NOT automatically
    deleted. Pointer eviction only removes the index entry.
```

### 11.5 Integration with Runtime Layers

RELAY sits between snapshots and the runtime hooks:

```
Session End
    │
    ├── write session_snapshot ───→ snapshot store
    │
    └── append RELAY pointer ────→ RELAY file
                                      │
Session Start                          │
    │                                  │
    └── read RELAY ────────────────────┘
            │
            └── load latest snapshot → inject via pre_llm_call
```

```yaml
relay_in_runtime_stack:
  pre_llm_call:
    input: "latest snapshot content"
    source: "RELAY → snapshot_location → snapshot file"
  post_llm_call:
    update: "append new pointer if snapshot was written"
    cleanup: "prune expired tasks if TTL exceeded"
  state_machine:
    maintain: "RELAY file integrity"
    counters: "turns_since_snapshot, session_count"
```

### 11.6 Schema

See [`schemas/relay_session_pointer.schema.yaml`](../schemas/relay_session_pointer.schema.yaml) for the full schema definition.
