# Glow Framework Dependency Policy

`glow-ops` depends on the framework repository `github.com/luaxlou/glow`.

## Versioning Rules

- Production/default dependency is pinned in `go.mod`:
  - `github.com/luaxlou/glow v1.0.0-beta.20`
- Do not use floating branches in `go.mod`.
- Upgrade only by explicit PR with full regression tests.

## Local Development

If developing both repos together, use a temporary local replace (do not commit):

```bash
go mod edit -replace github.com/luaxlou/glow=../glow
```

And revert before commit:

```bash
go mod edit -dropreplace github.com/luaxlou/glow
```

## Upgrade Checklist

1. Bump `github.com/luaxlou/glow` version in `go.mod`.
2. Run `go mod tidy`.
3. Run `go test ./...` and `go vet ./...`.
4. Verify starter-related integration paths in server/cli flows.
