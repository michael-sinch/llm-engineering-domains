# LLM Engineering Domains

Domain repository for the [llm-engineering-agent](https://github.com/michael-sinch/llm-engineering-agent) framework. Stores session state, assignment metadata, and cross-model handoff context outside of target repositories.

## Purpose

The llm-engineering-agent framework uses a session model where each development task gets a dedicated session directory containing:

- **`agent-assignment.md`** — Task definition, intake decisions, and current phase status
- **`session-context.md`** — Running decision log for context recovery and cross-model handoff

These files live here — separate from both the framework repo and the target code repos — to keep domain knowledge version-controlled without polluting either.

## Structure

```
<domain>/
  <team-or-repo>/
    sessions/
      <ticket-id>/
        agent-assignment.md
        session-context.md
```

### Current domains

- **`platform_identity/`** — Sinch Identity & Mailgun Platform services (gozer, user-management-bff, sinch-identity-auth0-terraform, etc.)

## Related

- [llm-engineering-agent](https://github.com/michael-sinch/llm-engineering-agent) — The framework that consumes these session files
