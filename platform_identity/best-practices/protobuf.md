# Best Practices: Protobuf and gRPC

**Last Updated:** March 13, 2026
**Audience:** LLM Agents, Developers
**Scope:** All platform_identity services using gRPC

---

## Table of Contents

- [Proto-First API Design](#proto-first-api-design)
- [Code Generation](#code-generation)
- [go_package Consistency](#go_package-consistency)
- [go.mod replace Directive Workflow](#gomod-replace-directive-workflow)

---

## Proto-First API Design

**Rule: Update `.proto` files before implementing service methods.**

The `.proto` definition is the contract. Implementation follows the contract — not the other way around.

1. Define or update the RPC method and message types in the `.proto` file
2. Run `buf generate` (or `make proto`) to regenerate Go code
3. Implement the service method to satisfy the generated interface
4. Write tests against the generated types

Never modify generated `.pb.go` files directly. All changes go through the `.proto` source.

---

## Code Generation

- Use `buf generate` as the canonical generation command — do not use `protoc` directly unless `buf` is unavailable
- Generated output must go to a single canonical location matching your module/import path
- After any `.proto` change, run generation, then `go build ./...` immediately to catch import or type errors before writing implementation code
- Pin tool versions (`buf`, `protoc-gen-go`, `protoc-gen-go-grpc`) in CI to prevent environment drift

---

## go_package Consistency

`option go_package` in the `.proto` file is authoritative for the Go import path of generated code.

When changing `go_package`:
1. Update the proto file
2. Regenerate all affected `.pb.go` files
3. Update all Go import statements in the codebase in the same commit
4. Do not leave partial imports pointing to the old path

Mismatched `go_package` and import paths cause build failures that are difficult to diagnose. Keep them in sync.

---

## go.mod replace Directive Workflow

When developing against an unpublished version of a dependency (e.g. a local Jefe change not yet merged):

```
// Uncomment for local development against unpublished jefe client
// replace github.com/mailgun/jefe => ../jefe
```

**Lifecycle:**

| Phase | Directive state |
| --- | --- |
| Local development | Uncommented (active) |
| Before committing | Commented out |
| After upstream merges and version is cut | Remove the line entirely |

**Rules:**

- Never commit an active (uncommented) replace directive — it will break CI/CD
- The commented form is documentation: it tells the next developer how to enable local cross-service development
- Evaluate CI/CD impact before any `go.mod` change — replace directives, version bumps, and new dependencies must be assessed for pipeline compatibility before the change is made
