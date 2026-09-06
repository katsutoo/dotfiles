# Go backend stack

Read the relevant sections when selecting a new backend stack or changing HTTP,
persistence, migrations, logging, startup, or shutdown. Resolve exact module and
tool versions first.

## Opinionated starter options

Use these only for a new backend application when its requirements fit. Third-party entries are reasonable choices, not language-wide defaults.

| Area | Option | Notes |
| --- | --- | --- |
| Toolchain | Mandatory minimum `go` directive; optional preferred `toolchain` directive | Account for `GOTOOLCHAIN` and automatic switching; do not rewrite directives incidentally |
| Formatting | `gofmt`; optionally `goimports` | `goimports` also applies Go formatting while organizing imports |
| HTTP | `net/http`; Chi when its routing features help | Keep handlers and middleware compatible with standard `http.Handler` |
| PostgreSQL | `pgx`; add `pgxpool` or `sqlc` when useful | Match connection and query tooling to the service's needs |
| Migrations | Existing deployment-owned system; Goose SQL is one option | Keep migrations versioned, ordered, reviewed, and deployment-owned |
| JSON | `encoding/json` | Use explicit request/response DTOs and stable tags |
| Logging | `log/slog` | Use structured service logs unless the repository has an established API |
| Validation | Explicit validation; optionally `go-playground/validator` | Add declarative validation when contract complexity justifies it |
| Password hashing | Argon2id | If the service stores passwords |
| IDs | Domain-specific; UUIDv7 when time ordering and locality help | IDs are identifiers, not authorization secrets |
| Date/time | `time` (stdlib) | Keep time zones explicit and consistent |
| Integration tests | Real isolated dependencies; optionally `testcontainers-go` | Use containers when realism justifies requiring a container runtime |
| Static analysis | `go vet`; optionally `staticcheck` | Run repository-configured analyzers |
| Vulnerability review | `govulncheck` | Run regularly and after dependency or toolchain changes |

If the repository already uses Echo, Gin, Fiber, GORM, Bun, or another established stack, stay consistent unless the user explicitly asks for a migration.

## HTTP, database, and security

- In application HTTP code, use `http.Status...` constants rather than numeric literals. Protocol tables and parser fixtures may use numbers when that is clearer.
- Configure `http.Server.ReadHeaderTimeout` and `IdleTimeout`, then choose `ReadTimeout` and `WriteTimeout` only after accounting for request bodies and streaming behavior.
- Reuse `http.Client` and its `Transport`; choose an end-to-end client timeout, per-request context deadlines, or both according to the operation.
- After a successful `Client.Do`, close `resp.Body` on every path. Consume it as required by the protocol and connection-reuse policy, and bound reads from untrusted peers.
- Shut servers down with `http.Server.Shutdown` and a bounded context, treat `http.ErrServerClosed` as expected, stop accepting work, and wait for owned work that must complete.
- Bound request bodies before decoding with `http.MaxBytesReader` or `http.MaxBytesHandler`, using endpoint-specific limits for JSON and uploads.
- Configure `httputil.ReverseProxy` with `Rewrite`, not the deprecated and insecure `Director`; explicitly decide whether and how to add trusted forwarding headers.
- Preserve standard-library limits on cookies and query parameters, handle parser failures explicitly, and do not relax compatibility controls for those limits or strict URL host parsing without a reviewed requirement.
- Keep response and error envelopes consistent within an API surface.
- The code coordinating an atomic use case owns the transaction boundary.
- Keep SQL in queries or repository code, not in handlers.
- Treat `sqlc` output as generated code: regenerate it, do not hand-edit it.
- Before schema or performance-sensitive query changes, load the matching database skill. Account for table size, lock behavior, deployment order, and overlapping application versions; prefer expand-and-contract changes and separate bounded backfills from deploy-time migrations.

## Chi and net/http

- Use Chi as a thin `net/http` router. Keep handlers, middleware, tests, and
  server configuration based on standard `http.Handler` contracts.
- Group routes by feature and mount subrouters where that improves ownership.
  Apply authentication, request limits, and resource-loading middleware at the
  narrowest route scope that enforces the policy.
- Middleware order is semantic. Test panic recovery, request IDs, real client IP,
  logging, timeout, compression, authentication, and body-limit interactions.
- Configure an explicit `http.Server`, signal-driven shutdown, finite drain
  timeout, and ownership for work that must finish after request admission stops.

## pgxpool and sqlc

- Create one `pgxpool.Pool` at startup, ping it before readiness, size it across
  all replicas, monitor acquire waits and saturation, and close it on shutdown.
- Write reviewed SQL and let sqlc generate typed query code. Treat generated Go
  as output: change SQL or sqlc configuration, regenerate with the pinned tool,
  and inspect the diff.
- Keep sqlc interfaces narrow at the consuming operation when transactions or
  tests need substitution. Do not wrap every generated method in boilerplate.
- Let the application operation own the transaction. Pass the transaction-bound
  generated query set through the operation, keep transactions short, and always
  commit or roll back explicitly.

## Row iteration

- With `database/sql` or pgx row iterators, close rows when abandoning iteration,
  handle every scan failure, and check `rows.Err()` after the loop. A false
  `Next()` can indicate either exhaustion or an iteration error; do not return
  partial results as success when the contract requires the complete result.
- Follow the selected driver's close and error-reporting contract, including
  batch cleanup. See [database/sql Rows](https://pkg.go.dev/database/sql#Rows).

## Goose

- Prefer SQL migrations unless a migration genuinely needs Go behavior. Keep
  files ordered, immutable after release, and safe for overlapping application
  versions.
- Run migrations once as a deployment step, not in every server process. Review
  locks, transaction support, rollback policy, and large backfills separately.
- Use timestamp versions during concurrent development only if the repository's
  release process normalizes or safely orders them. Preserve established policy.

## slog

- Construct the logger and handler at startup. Use JSON where machine ingestion
  needs it and pass a logger or scoped logger through explicit dependencies.
- Use stable attribute keys and context-aware methods when request or trace
  context is available. Redact secrets with boundary-owned values or handlers.
- Log an operational failure once where it is handled or terminates work. Avoid
  eager expensive values on disabled log paths and custom wrappers that report
  the wrong source location.

## Verification and sources

- Test the full router with `httptest`, SQL against a migrated disposable
  database, generated-code drift, migration up/down policy, readiness, and
  graceful shutdown.
- Chi: `https://go-chi.io/`
- pgxpool: `https://pkg.go.dev/github.com/jackc/pgx/v5/pgxpool`
- sqlc: `https://docs.sqlc.dev/`
- Goose: `https://pressly.github.io/goose/`
- slog: `https://pkg.go.dev/log/slog`
