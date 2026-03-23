# Policy: Authentication and Identity

**Last Updated:** March 13, 2026
**Audience:** LLM Agents, Developers
**Scope:** All platform_identity services integrating with Auth0

---

## Table of Contents

- [Auth0 Identifier Guidance](#auth0-identifier-guidance)
- [JWT and Session Validation](#jwt-and-session-validation)
- [Management API Access](#management-api-access)
- [Credential Handling](#credential-handling)

---

## Auth0 Identifier Guidance

**Rule: Auth0 user IDs and internal GUIDs are not interchangeable. Never substitute one for the other.**

Auth0 exposes its own user identifier (format: `auth0|<alphanumeric>`). This is distinct from the application's internal GUID and cannot be used as a drop-in replacement.

**Lookup sequence for Auth0 operations:**

1. Identify the user by their internal application identifier or email
2. Retrieve the Auth0 user ID for that account (via Management API or stored mapping)
3. Call the Auth0 Management API using the Auth0 user ID

**Storage requirement:**

When synchronizing users across systems, store both the internal GUID and the Auth0 user ID to enable reliable cross-references.

**Shell gotcha:**

Auth0-style IDs contain `|` (pipe). When passing them as CLI arguments, always quote:

```bash
# Wrong — shell interprets | as pipe
--user-id auth0|abc123

# Correct
--user-id 'auth0|abc123'
```

---

## JWT and Session Validation

- All endpoints must validate the JWT before accessing any resource
- Extract claims from the validated token, not from request parameters
- `email` and user identity claims must come from `claims["email"]`, not from query params or request body — check sibling handlers in the same package for the established pattern before writing a new one
- Do not defer auth validation to downstream services

---

## Management API Access

- Management API credentials (client ID + secret) are OAuth2 machine-to-machine credentials stored in Vault
- Local development uses `config/dev.yaml` populated from Vault staging secrets
- Credentials must never be hardcoded or committed
- The Management API client is the boundary for all Auth0 operations — do not construct raw Auth0 API calls from services that have access to the typed client

---

## Credential Handling

- All secrets are stored in Vault and loaded via config at startup
- Services validate credential presence at startup and fail fast if missing
- Credentials are never logged (see [logging policy](logging.md))
- OAuth2 token acquisition is handled by the Management API client — consuming services do not manage token lifecycle directly
