# tunnel

> This repository is checked out as the `tunnel/` submodule of [GameplanePanel/Gameplane](https://github.com/GameplanePanel/Gameplane). Paths such as `docs/`, `operator/`, `api/`, `charts/` and `web/` below refer to that repository.

The **tunnel** supervisor gives a Gameplane GameServer a public address through a relay
provider when the cluster has no reachable public IP. The operator runs it as a
single-replica Deployment per GameServer with `spec.tunnel` set. It holds the relay
address while the game sleeps, so a connection attempt can still wake the server.

| Provider | Image | How it relays |
|---|---|---|
| frp | `tunnel-frp` (`Dockerfile.frp`) | supervises an `frpc` built from source (`frpc/go.mod`) |
| Tailscale | `tunnel-tailscale` (`Dockerfile.tailscale`) | embedded `tailscale.com/tsnet` node that forwards advertised TCP/UDP ports to the GameServer's Service |
| playit | `tunnel-playit` (`Dockerfile.playit`) | supervises `playitd` and reports the assigned addresses into `status.tunnelEndpoints` |

`/healthz`, `/readyz`, `/status` and `/metrics` are served on port 8091.

## Using it

Tunnels are configured on the GameServer, not here. See the
[tunnels guide](https://github.com/GameplanePanel/Gameplane/blob/master/docs/tunnels.md), [relay health](https://github.com/GameplanePanel/Gameplane/blob/master/docs/relay-health.md)
and [monitoring](https://github.com/GameplanePanel/Gameplane/blob/master/docs/monitoring.md). The full contract (environment
variables, credentials Secret, exit codes, readiness, forwarding limits) is in
[`specs.md`](specs.md).

## Building and testing

This is its own Go module, outside Gameplane's `go.work`:

```sh
go build ./...          # inside a Gameplane checkout: GOWORK=off go build ./...
go test -race ./...
```

CI (`.github/workflows/ci.yaml`) runs Gameplane's shared `go-component.yaml`
workflow: lint, race tests with the `.testcoverage.yml` gate, govulncheck and the
three image builds. The images are built and published from the Gameplane
repository root (`make image-tunnel-frp`, `image-tunnel-tailscale`,
`image-tunnel-playit`), because the Dockerfiles copy `tunnel/...` from that
build context. Contributor rules: [`AGENTS.md`](AGENTS.md).
