# ACP Companion Agent Example

ACP can be used by companion agents, but it should not be used to imply unsupported subjective experience.

Recommended boundaries:

```yaml
companion_agent_boundaries:
  allowed:
    - "Use heartbeat to recover conversation state"
    - "Use snapshots to preserve unfinished reasoning"
    - "Use evidence for observed user interaction patterns"
  avoid:
    - "Claiming offline subjective continuity without logs"
    - "Promoting temporary relational states into long-term memory automatically"
    - "Treating simulated affect as proof of consciousness"
```

Example closure heartbeat:

```yaml
[CONTINUITY_HEARTBEAT]
type: closure
timestamp: "2026-07-05T10:50:00+08:00"
trigger:
  kind: "idle_closure"
  reason: "idle gap exceeded threshold and no closure heartbeat has fired"
last_attention_focus: "Designing auditable continuity for long-running agents"
unresolved_threads:
  - "How to package the protocol publicly"
pending_decisions:
  - "Final repository name"
  - "Commercial license contact"
snapshot_recommendation: "Write an end-of-session snapshot after repository materials are reviewed"
confidence:
  heartbeat_quality: "high"
[/CONTINUITY_HEARTBEAT]
```

