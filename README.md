# Glow Ops

`glow-ops` is the operations runtime repository.

## Scope

This repo contains control-plane and operations lifecycle components:

- `glow-server`
- `glow-cli`
- runtime lifecycle orchestration (start/stop/restart/health/rollback)
- process supervision, ingress automation, resource binding, state/config management

## Framework Dependency

`glow-ops` depends on the `glow` framework repository for starter packages used by runtime internals (for example sqlite/http starters).

## Development

```bash
go test ./...
```

Operational manuals are in `docs/`.
