# ACP Codex Lite Example

Codex-like coding agents usually need engineering checkpoints more than persona continuity.

Recommended subset:

```yaml
codex_continuity_lite:
  enabled_for:
    - long_running_task
    - multi_file_change
    - interrupted_work
    - research_then_implementation
    - user_returns_after_long_gap
  snapshots:
    - task_state
    - files_touched
    - commands_run
    - assumptions
    - blockers
    - next_action
  heartbeat:
    default: "off"
    use_only_as:
      - "checkpoint before risky edit"
      - "checkpoint before final answer"
      - "checkpoint after major implementation decision"
  memory_promotion:
    default: "off"
  emotional_or_persona_state:
    disabled: true
```

Example snapshot:

```yaml
[SESSION_SNAPSHOT]
timestamp: "2026-07-05T10:45:00+08:00"
task_anchor: "Implement ACP public repository package"
attention_focus:
  - "Sanitize private material"
  - "Choose a commercial-use-restricted license"
active_memories:
  - name: "User licensing preference"
    reason_used: "User requested commercial users must pay"
unfinished_reasoning:
  - "Final repository name is not confirmed"
interaction_mode:
  observed: "The user is asking for publication-ready deliverables"
  evidence:
    - "The user asked for GitHub-ready materials"
next_best_action:
  - "Review package and replace placeholders"
confidence:
  snapshot_quality: "high"
[/SESSION_SNAPSHOT]
```

