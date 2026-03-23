# Best Practices: Go Error Handling

**Last Updated:** March 13, 2026
**Audience:** LLM Agents, Developers
**Scope:** All platform_identity Go services

---

## Table of Contents

- [Error Wrapping with pkg-errors](#error-wrapping-with-pkg-errors)
- [Error Propagation Rules](#error-propagation-rules)
- [HTTP Error Translation](#http-error-translation)

---

## Error Wrapping with pkg-errors

Use `github.com/pkg/errors` for error wrapping. This preserves stack traces and enables root-cause analysis in logs.

```go
import "github.com/pkg/errors"

// Wrap at call sites — add context without obscuring the original error
if err := db.Find(id); err != nil {
    return errors.Wrap(err, "finding user record")
}

// Wrapf for formatted context
return errors.Wrapf(err, "creating enrollment ticket for user %s", userID)
```

**Rules:**

- Wrap errors at every layer boundary (handler → service → client)
- Add context that identifies the operation, not just the location
- Do not double-wrap: if a called function already wraps, the caller adds its own context on top — do not re-wrap with the same message

---

## Error Propagation Rules

- **Never swallow errors.** If a function returns an error, either handle it or wrap and return it.
- **Never return a generic `InternalError` for a known error type.** If `translateError` produces a typed error, pass it through — do not wrap it in `InternalError{}`.
- At service boundaries (HTTP handlers, gRPC interceptors), translated errors must reach the caller with their original type and detail intact.

```go
// Wrong — swallows the translated error
ticket, err := c.mgr.CreateEnrollmentTicket(ctx, userID, params)
if err != nil {
    return nil, httpapi.InternalError{}
}

// Correct — passes translated error through
ticket, err := c.mgr.CreateEnrollmentTicket(ctx, userID, params)
if err != nil {
    return nil, err
}
```

---

## HTTP Error Translation

Services that proxy third-party APIs (e.g. Jefe proxying Auth0) must implement a `translateError` function that maps upstream error codes to typed httpapi errors.

- `translateError` is called once, at the service boundary
- Its output is returned directly to the caller — not re-wrapped
- `InternalError` is reserved for genuinely unexpected conditions (panics, nil pointers, unrecognized error shapes) that have no upstream detail to surface
