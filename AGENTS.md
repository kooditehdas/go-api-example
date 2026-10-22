# Agent instructions

These rules apply to every AI agent working in this repository (implementer and reviewer alike) and to humans.

## Project

An HTTP JSON API in Go, using only the standard library (`net/http` with Go 1.22+ routing patterns).

- `cmd/server/main.go` starts the HTTP server with timeouts and graceful shutdown. Keep it small; no business logic.
- `internal/server/server.go` builds the handler: `New()` registers every route on one `http.ServeMux`. It never listens on a port, so tests can use it directly.
- One file per resource in `internal/server/` (for example `health.go`), registered in `New()` with a method pattern such as `"GET /health"`.
- Tests sit next to the code (`*_test.go`) and use `httptest`, never a real port or network.

## Commands

| Purpose | Command                   |
| ------- | ------------------------- |
| Install | `go mod download`         |
| Format  | `gofmt -w .`              |
| Lint    | `golangci-lint run`       |
| Vet     | `go vet ./...`            |
| Test    | `go test -race ./...`     |
| Build   | `go build ./...`          |

Lint, vet, test and build must all pass before work is considered done. Lint settings are in `.golangci.yml` (it also checks `gofmt`/`goimports` formatting).

## Conventions

- Responses are JSON written with `writeJSON`. Errors use `errorBody{Error: "<message>"}` with a matching status code. Unknown paths and unsupported methods return the JSON 404 from `notFound`.
- Validate request input at the handler boundary: decode with `json.Decoder`, reject unknown or invalid fields with 400, and limit body size with `http.MaxBytesReader`.
- Return errors instead of panicking; wrap them with `fmt.Errorf("...: %w", err)` and compare with `errors.Is`/`errors.As`.
- Use `log/slog` for logging. Never log secrets or full request bodies.
- Tests: table-driven where there are several cases, `t.Helper()` in helpers, and both the success path and at least one failure path for every route.
- Standard library first. A new module dependency needs a clear reason stated in the PR.
- Exported identifiers get doc comments. Keep packages under `internal/` unless something truly must be importable.

## Constraints

- Never commit secrets, tokens or real credentials. Configuration comes from environment variables documented in `.env.example`.
- Never weaken, skip (`t.Skip`) or delete tests to make a check pass, and never silence linters with `//nolint` without an explaining comment.
- Do not change unrelated code, formatting or files.
- Do not edit `.github/`, `factory.yaml` or `.factory/` unless the task explicitly asks for it.

## Definition of done

1. The acceptance criteria in the issue are met.
2. Tests cover the change.
3. Lint, vet, test and build pass locally.
4. The change is summarized with any remaining risks or follow-ups.
