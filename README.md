# Glow Ops

`glow-ops` 是运维与控制面仓库，提供 `glow-server` 与 `glow-cli`。

## 快速决策：我该看哪个仓库？

| 你的目标 | 去哪个仓库 |
|---|---|
| 写业务代码、接入 SDK/starter | [`glow`](https://github.com/luaxlou/glow) |
| 做发布与运维编排、控制面治理 | [`glow-ops`](https://github.com/luaxlou/glow-ops) |

## 双仓关系图

```text
+-------------------+         depends on starters         +-------------------+
|      glow-ops     | ----------------------------------> |       glow        |
| (server/cli/ops)  |                                     | (starter/sdk)     |
+-------------------+                                     +-------------------+
```

## 这个仓库解决什么问题

如果你需要管理应用运行生命周期与运维能力（而不是写业务代码），你应该使用这个仓库：

- 应用生命周期编排（start/stop/restart/health/rollback）
- 进程托管与日志管理
- 资源绑定、状态管理
- 控制面 API 与 CLI

## 核心组件

- `cmd/glow-server`：服务端入口
- `cmd/glow`：CLI 入口
- `internal/apiserver`：控制面 HTTP API
- `internal/manager`：运行与编排核心逻辑
- `internal/configmanager` / `internal/statemanager`：配置与状态持久化

## 与框架仓关系

`glow-ops` 依赖框架仓 [`glow`](https://github.com/luaxlou/glow) 的 starter 能力（如 HTTP/SQLite 适配）。

依赖策略见：[`docs/framework-dependency-policy.md`](./docs/framework-dependency-policy.md)

## 文档

- [`docs/server_manual.md`](./docs/server_manual.md)
- [`docs/cli_manual.md`](./docs/cli_manual.md)

## 开发与验证

```bash
go test ./...
go vet ./...
```

## 相关仓库

- 应用框架：[`glow`](https://github.com/luaxlou/glow)
