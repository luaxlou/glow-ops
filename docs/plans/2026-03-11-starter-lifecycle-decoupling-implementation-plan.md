# Starter-Lifecycle Decoupling Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Decouple operations lifecycle orchestration from Glow starters so starters remain application-facing capability adapters while runtime lifecycle is centralized in a dedicated orchestration layer.

**Architecture:** Keep `starter/*` strictly focused on app runtime integrations (config/http/db clients), and move lifecycle orchestration (`start/stop/restart/health/rollback`) into `internal/lifecycle`. API handlers and manager facade should call lifecycle use cases rather than embedding orchestration logic. Update docs/specs in lockstep per task to keep user-facing semantics consistent.

**Tech Stack:** Go 1.24+, Gin, Glow starters, internal manager/apiserver/configmanager modules, OpenSpec docs, Go test/vet/gofmt.

---

### Task 1: Define Lifecycle Boundary Contract

**Files:**
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/contracts.go`
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/model.go`
- Test: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/contracts_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/sdk_manual.md`

**Step 1: Write the failing test**

```go
func TestLifecycleContract_NoStarterDependency(t *testing.T) {
	// compile-time assertion that lifecycle contracts only reference internal/domain abstractions
}
```

**Step 2: Run test to verify it fails**

Run: `go test ./internal/lifecycle -v`
Expected: FAIL (package/contracts missing)

**Step 3: Write minimal implementation**

```go
type ProcessSupervisor interface { Start(...); Stop(...); Restart(...) }
type HealthVerifier interface { Verify(...) error }
type RollbackPlanner interface { Plan(...) (..., error) }
```

**Step 4: Run test to verify it passes**

Run: `go test ./internal/lifecycle -v`
Expected: PASS

**Step 5: Update docs for boundary**

- In `docs/sdk_manual.md`, explicitly state starters do not orchestrate lifecycle.

**Step 6: Commit**

```bash
git add internal/lifecycle docs/sdk_manual.md
git commit -m "refactor(lifecycle): define starter-independent lifecycle contracts"
```

### Task 2: Introduce Lifecycle Service for Start/Stop/Restart

**Files:**
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/service.go`
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/service_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/internal/manager/manager.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/server_manual.md`

**Step 1: Write the failing test**

```go
func TestLifecycleService_StartStopRestartFlow(t *testing.T) {
	// fake supervisor + fake state store
	// expect deterministic state transitions and errors surfaced
}
```

**Step 2: Run test to verify it fails**

Run: `go test ./internal/lifecycle -v -run StartStopRestart`
Expected: FAIL (`Service` missing)

**Step 3: Write minimal implementation**

```go
func (s *Service) Start(ctx context.Context, req StartRequest) error
func (s *Service) Stop(ctx context.Context, req StopRequest) error
func (s *Service) Restart(ctx context.Context, req RestartRequest) error
```

**Step 4: Rewire manager facade**

- Keep `manager.StartApp/StopApp/Restart` as thin calls into lifecycle service.
- Remove orchestration logic duplication from manager where possible.

**Step 5: Run test to verify it passes**

Run: `go test ./internal/lifecycle -v -run StartStopRestart && go test ./internal/manager -v`
Expected: PASS

**Step 6: Update docs**

- `docs/server_manual.md`: lifecycle ownership belongs to runtime layer, not starter.

**Step 7: Commit**

```bash
git add internal/lifecycle internal/manager docs/server_manual.md
git commit -m "feat(lifecycle): centralize start stop restart orchestration"
```

### Task 3: Extract Health and Verification Flow from API Handlers

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/internal/apiserver/server.go`
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/health.go`
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/health_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/server_manual.md`

**Step 1: Write the failing test**

```go
func TestHealthVerifier_MapsProcessStateAndProbeResult(t *testing.T) {}
```

**Step 2: Run test to verify it fails**

Run: `go test ./internal/lifecycle -v -run HealthVerifier`
Expected: FAIL

**Step 3: Write minimal implementation**

```go
type HealthResult struct { Status string; Detail string }
func (h *HealthVerifierImpl) Verify(ctx context.Context, appName string) (HealthResult, error)
```

**Step 4: Rewire API handler**

- API handler should call lifecycle verifier instead of mixing direct manager/statemanager/process inspection logic.

**Step 5: Run tests**

Run: `go test ./internal/lifecycle -v && go test ./internal/apiserver -v`
Expected: PASS

**Step 6: Update docs**

- Clarify health check path and lifecycle verification path in `docs/server_manual.md`.

**Step 7: Commit**

```bash
git add internal/lifecycle internal/apiserver docs/server_manual.md
git commit -m "refactor(apiserver): use lifecycle health verifier"
```

### Task 4: Add Rollback Orchestration in Lifecycle Layer

**Files:**
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/rollback.go`
- Create: `/Users/john/workspace/luaxlou/glow/internal/lifecycle/rollback_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/internal/manager/manager.go`
- Modify: `/Users/john/workspace/luaxlou/glow/openspec/specs/process-governance/spec.md`

**Step 1: Write the failing test**

```go
func TestRollback_UsesSavedRevisionPlanAndSupervisor(t *testing.T) {}
```

**Step 2: Run test to verify it fails**

Run: `go test ./internal/lifecycle -v -run Rollback`
Expected: FAIL

**Step 3: Write minimal implementation**

```go
func (s *Service) Rollback(ctx context.Context, req RollbackRequest) error
```

**Step 4: Connect manager façade**

- Manager rollback entrypoint (if present) becomes lifecycle call.

**Step 5: Run tests**

Run: `go test ./internal/lifecycle -v -run Rollback && go test ./internal/manager -v`
Expected: PASS

**Step 6: Update spec doc**

- `openspec/specs/process-governance/spec.md`: update source of truth for rollback ownership.

**Step 7: Commit**

```bash
git add internal/lifecycle internal/manager openspec/specs/process-governance/spec.md
git commit -m "feat(lifecycle): add rollback orchestration"
```

### Task 5: Remove Lifecycle Semantics from Starter HTTP

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/starter/glowhttp/server.go`
- Create: `/Users/john/workspace/luaxlou/glow/starter/glowhttp/server_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/sdk_manual.md`

**Step 1: Write the failing test**

```go
func TestGlowHTTP_ServerProvidesTransportOnly_NoLifecycleStateMachine(t *testing.T) {}
```

**Step 2: Run test to verify it fails**

Run: `go test ./starter/glowhttp -v`
Expected: FAIL

**Step 3: Write minimal implementation**

- Provide explicit server object lifecycle (`NewServer`, `Run`, `Shutdown`) as transport primitive only.
- Remove hidden lifecycle semantics from starter globals where feasible.

**Step 4: Run tests**

Run: `go test ./starter/glowhttp -v`
Expected: PASS

**Step 5: Update docs**

- `docs/sdk_manual.md`: starter/glowhttp is an HTTP transport helper only.

**Step 6: Commit**

```bash
git add starter/glowhttp docs/sdk_manual.md
git commit -m "refactor(starter): limit glowhttp to transport concerns"
```

### Task 6: Remove Lifecycle Semantics from Starter Config/DB/Redis

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/starter/glowconfig/config.go`
- Modify: `/Users/john/workspace/luaxlou/glow/starter/glowmysql/mysql.go`
- Modify: `/Users/john/workspace/luaxlou/glow/starter/glowredis/redis.go`
- Modify: `/Users/john/workspace/luaxlou/glow/starter/glowsqlite/sqlite.go`
- Create: `/Users/john/workspace/luaxlou/glow/starter/glowmysql/mysql_test.go`
- Create: `/Users/john/workspace/luaxlou/glow/starter/glowredis/redis_test.go`
- Create: `/Users/john/workspace/luaxlou/glow/starter/glowsqlite/sqlite_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/sdk_manual.md`

**Step 1: Write failing tests**

```go
func TestStarterConfig_ReturnsExplicitErrorOnMissingConfig(t *testing.T) {}
func TestStarterMySQL_InitAndReloadContract(t *testing.T) {}
func TestStarterRedis_InitAndReloadContract(t *testing.T) {}
func TestStarterSQLite_InitAndSchemaContract(t *testing.T) {}
```

**Step 2: Run tests to verify failure**

Run: `go test ./starter/... -v`
Expected: FAIL

**Step 3: Write minimal implementation**

- Add explicit error-returning APIs where needed.
- Ensure starters expose capability contracts only (no runtime orchestration assumptions).

**Step 4: Run tests**

Run: `go test ./starter/... -v`
Expected: PASS

**Step 5: Update docs**

- `docs/sdk_manual.md`: capability-only semantics for all starters.

**Step 6: Commit**

```bash
git add starter docs/sdk_manual.md
git commit -m "refactor(starter): enforce capability-only contracts"
```

### Task 7: Rewire API Server to Lifecycle Use Cases

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/internal/apiserver/server.go`
- Modify: `/Users/john/workspace/luaxlou/glow/internal/apiserver/server_test.go`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/server_manual.md`
- Modify: `/Users/john/workspace/luaxlou/glow/README.md`

**Step 1: Write failing tests**

```go
func TestAPIServer_StartStopRestartDelegateToLifecycle(t *testing.T) {}
```

**Step 2: Run tests to verify failure**

Run: `go test ./internal/apiserver -v`
Expected: FAIL

**Step 3: Write minimal implementation**

- Inject/use lifecycle service in handlers.
- Keep route surface stable but remove orchestration from handler methods.

**Step 4: Run tests**

Run: `go test ./internal/apiserver -v`
Expected: PASS

**Step 5: Update docs**

- `README.md` and `docs/server_manual.md` to match new ownership model.

**Step 6: Commit**

```bash
git add internal/apiserver docs/server_manual.md README.md
git commit -m "refactor(apiserver): delegate runtime orchestration to lifecycle service"
```

### Task 8: OpenSpec and Manual Consistency Pass

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/openspec/specs/process-governance/spec.md`
- Modify: `/Users/john/workspace/luaxlou/glow/openspec/specs/system-initialization/spec.md`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/sdk_manual.md`
- Modify: `/Users/john/workspace/luaxlou/glow/docs/server_manual.md`
- Modify: `/Users/john/workspace/luaxlou/glow/README.md`

**Step 1: Write failing consistency check**

- Add/execute a doc consistency checklist script or manual grep checklist for terms:
  - `starter` must not claim lifecycle orchestration
  - `lifecycle/runtime` must own start/stop/restart/rollback

**Step 2: Run check to verify it fails before updates**

Run: `rg -n "starter|lifecycle|orchestration|start|rollback" README.md docs openspec/specs`
Expected: detect inconsistent statements

**Step 3: Update docs/specs**

- Align terminology and ownership statements.

**Step 4: Re-run consistency check**

Run: `rg -n "starter|lifecycle|orchestration|start|rollback" README.md docs openspec/specs`
Expected: no contradictions

**Step 5: Commit**

```bash
git add README.md docs openspec/specs
git commit -m "docs(spec): align starter and lifecycle ownership semantics"
```

### Task 9: Final Verification Before Completion

**Files:**
- Modify: `/Users/john/workspace/luaxlou/glow/CHANGELOG.md`

**Step 1: Run full tests**

Run: `go test ./... -v`
Expected: PASS

**Step 2: Run static checks**

Run: `go vet ./...`
Expected: PASS

**Step 3: Run formatting**

Run: `gofmt -w $(find . -name '*.go')`
Expected: no formatting drift after second run

**Step 4: Re-run tests**

Run: `go test ./... -v`
Expected: PASS

**Step 5: Update changelog**

- Record starter-lifecycle decoupling and doc/spec alignment.

**Step 6: Commit**

```bash
git add CHANGELOG.md $(find . -name '*.go')
git commit -m "chore: finalize starter lifecycle decoupling"
```
