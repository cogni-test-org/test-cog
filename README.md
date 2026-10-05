# node-template

Node-at-root template for a **Cogni full-app submodule node** — the canonical single node, minted via GitHub generate-from-template and added as a git submodule at `nodes/<slug>/` in the operator monorepo (`Cogni-DAO/cogni`).

Seeded from `Cogni-DAO/cogni:nodes/node-template/` and projected to a
node-at-repo-root build surface.

- **Node-at-root** layout (`app/`, `graphs/`, `packages/`, `.cogni/`) so it mounts cleanly at `nodes/<slug>/` when added as a submodule.
- `.cogni/secrets-catalog.yaml` + `k8s/external-secrets/` are intentionally absent (bug.5086 Part D: a node inherits baseline secrets via the substrate; declares its own only when it has unique ones).
- The root workspace includes the required private `@cogni/*` package closure so a generated child repo can install, typecheck, build, and push its own image without the operator monorepo.
- `.github/workflows/ci.yaml` builds the app image, pushes to the repo-owned GHCR package (`ghcr.io/<owner>/<repo>-node`), pushes PR images as `pr-<number>-<headSha>`, and pushes `main` images as `sha-<childSha>`.
- The deploy/infra plane is intentionally absent: no candidate-flight, preview/prod promote, provision-env, Argo/AppSet, or parent infra workflows live here.

## Local checks

```bash
pnpm install --frozen-lockfile
pnpm check
docker build --target runner -t cogni-node-template:local .
```

## Node developer guides

- `docs/guides/contributing-to-cogni.md` — node contribution loop and ownership boundaries.
- `docs/guides/new-node-styling.md` — first-pass rebrand for a minted node.
- `docs/guides/add-secret.md` — node-owned secret declarations and typed consumption.
- `docs/guides/contribute-knowledge.md` — when reusable findings should become durable node knowledge.

## Conductor workspaces

This template ships a shared Conductor setup entrypoint:

- `conductor.json`
- `scripts/conductor-worktree-setup.sh`

When a node repo minted from this template is added as a Conductor project,
Conductor runs the setup script from the new workspace. The script refreshes
`origin/main`, resolves this node's canonical main checkout from Conductor or
git worktree metadata (override with `COGNI_NODE_AUTH_ROOT`), ensures that
checkout has this node's `COGNI_NODE_API_KEY`, and symlinks `.env.cogni` and
`.local-auth` from it. On local Conductor it also installs the stable user-level
Codex cognition presenter, then installs dependencies, builds package
declarations, and writes
`.context/conductor-setup.json` as proof that setup ran.

For an unusual checkout layout where Conductor cannot resolve the node's main
checkout automatically:

```bash
export COGNI_NODE_AUTH_ROOT="/path/to/this-node-main-checkout"
```

The operator consumes the pushed digest and owns URL/DNS/deployment state.
