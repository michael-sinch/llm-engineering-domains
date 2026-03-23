# Policy: Logging Standards

**Last Updated:** March 13, 2026
**Audience:** LLM Agents, Developers
**Scope:** All platform_identity services

---

## Table of Contents

- [PII Handling in Logs](#pii-handling-in-logs)
- [Sensitive Data — Never Log](#sensitive-data--never-log)
- [Structured Logging](#structured-logging)
- [Log Levels](#log-levels)

---

## PII Handling in Logs

**Rule: PII must never appear in log output in raw form. Hash it before logging.**

Personally identifiable information includes:

- Email addresses
- Phone numbers
- Full names
- User IDs that map directly to a person (hash or truncate)
- Any field marked PII in the data model

**Pattern:**

```go
// Wrong
log.WithField("email", email).Info("processing request")

// Correct — hash before logging
log.WithField("email_hash", hashForLogging(email)).Info("processing request")
```

A `hashForLogging` helper must be used consistently. It should produce a stable, one-way hash (e.g. SHA-256 hex, first 8 chars) that allows log correlation without exposing the raw value.

---

## Sensitive Data — Never Log

The following must never appear in any log line, even hashed:

- Passwords or password hashes
- OAuth tokens, JWTs, API keys, client secrets
- Session cookies or session tokens
- MFA codes or backup codes

If a value is a credential, omit it entirely from logs. Log its presence or absence, not its value.

```go
// Wrong
log.WithField("token", token).Debug("auth token received")

// Correct
log.WithField("token_present", token != "").Debug("auth token received")
```

---

## Structured Logging

All services must use structured logging (key-value pairs), not string interpolation.

- Use the established logger for the service (do not introduce a new logger)
- Attach request context (user ID hash, request ID, trace ID) at the handler level and propagate via `context.Context`
- Log at the correct level — see table below

```go
// Wrong
log.Infof("user %s logged in from %s", userID, ip)

// Correct
log.WithField("user_id", userID).WithField("ip", ip).Info("user login")
```

---

## Log Levels

| Level | When to use |
| --- | --- |
| `DEBUG` | Detailed internal state during development; must not appear in production by default |
| `INFO` | Normal operational events — request received, resource created |
| `WARN` | Recoverable anomalies — upstream returned unexpected status, retrying |
| `ERROR` | Operation failed; requires attention but service is still running |
| `FATAL` | Service cannot continue; used only at startup for unrecoverable config errors |

Agent rule: Default to `INFO` for happy-path events and `WARN` for handled upstream errors. Reserve `ERROR` for failures that degrade service behavior.
