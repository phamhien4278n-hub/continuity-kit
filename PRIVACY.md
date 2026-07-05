# Privacy and Data Handling

ACP is a protocol. It does not require private conversation data to be published.

Before publishing examples, remove:

- Personal names.
- Private relationship terms.
- Raw user conversations.
- Private memory files.
- API keys, local paths, tokens, and credentials.
- Emotional-system internals unless intentionally released.

Recommended public example style:

```yaml
safe_example:
  task_anchor: "Designing an auditable continuity protocol"
  user_mode_observed:
    description: "The user moved from concept discussion to implementation planning"
    evidence:
      - "The user asked for a public repository package"
```

Avoid:

```yaml
unsafe_example:
  task_anchor: "Private relationship discussion with named person"
  evidence:
    - "Raw private quote from user"
```

## Data Retention

If ACP is used in production, disclose:

- What is logged.
- How long logs are retained.
- Who can access logs.
- How users can request deletion.
- Whether logs are used for training or evaluation.

