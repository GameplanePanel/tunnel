# tunnel — AI assistant guide

This repository holds the relay client supervisor for frp, Tailscale (embedded tsnet) and playit tunnels. It is a split component of [GameplanePanel/Gameplane](https://github.com/GameplanePanel/Gameplane), checked out there as the `tunnel/` submodule. What it does and its contracts: [`specs.md`](specs.md).

## How it fits into Gameplane

- **Own Go module**, not part of Gameplane's `go.work`. Build and test from this repository's root: `go build ./...` (inside a Gameplane checkout, prefix `GOWORK=off`).
- **CI** is `.github/workflows/ci.yaml`, a thin caller of Gameplane's shared [`go-component.yaml`](https://github.com/GameplanePanel/Gameplane/blob/master/.github/workflows/go-component.yaml), pinned by SHA: lint with Gameplane's root `.golangci.yml` and `.hadolint.yaml` (no copy here), race tests on amd64 and arm64 with the `.testcoverage.yml` gate, govulncheck, and an image build. Dependabot runs in this repository.
- **Images** `tunnel-frp`, `tunnel-tailscale` and `tunnel-playit` (`Dockerfile.frp`, `Dockerfile.tailscale`, `Dockerfile.playit`, built by `make image-tunnel-frp|tailscale|playit`) are built and published from the Gameplane repository root: the Dockerfiles copy `tunnel/...` from that build context.
- **Callers live in Gameplane:** paths such as `operator/`, `api/`, `charts/`, `docs/` and `web/` in `specs.md` and code comments refer to that repository.
- **Shipping a change:** PR to `master` here → after it merges, bump the `tunnel` pointer in Gameplane (`git -C tunnel checkout <merged sha on master>`, `git add tunnel`) in a Gameplane PR. A user-visible change gets its `changelog/unreleased/` entry in that Gameplane PR.

## Rules

Gameplane's [`AGENTS.md`](https://github.com/GameplanePanel/Gameplane/blob/master/AGENTS.md) core rules apply here. The ones that matter most in this repository:

- **CI verifies:** locally only compile checks (`go build ./...`); never run test or lint suites locally.
- **Fix, don't silence:** no `//nolint` and no per-repository `.golangci.yml`; `_test.go` files are exempt from `errcheck`/`gosec`/`unparam` through the shared config.
- **Go errors:** wrap with `%w`.
- **Tests:** never delete or weaken a test without the maintainer's explicit sign-off; fix the code.
- **Git:** signed commits (`git commit -s`), conventional prefixes (`feat:`, `fix:`, `chore:`, `docs:` …), one branch per unit of work, never amend pushed commits; delete the branch after merge.
- **Docs:** keep `specs.md` in step with behaviour changes in the same PR.
