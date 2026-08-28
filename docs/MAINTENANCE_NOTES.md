# Maintenance Notes: Dependency & Tooling Drift

Notes on the current state for a few open maintenance chores, so whoever picks up the
code change doesn't have to re-derive this context. These are notes only — none of the
three items below have been implemented here.

## zod minor bump (issue #1194)

- Currently pinned in [package.json](../package.json) at `"zod": "^3.24.1"`.
- The caret range already allows minor/patch updates on `pnpm install` — the
  `package-lock.json` / `pnpm-lock.yaml` entry is what's actually behind, not the
  `package.json` range itself. Bumping means running the install and committing the
  updated lockfile, then confirming the `package.json` range still covers the resolved
  version (or bumping it explicitly if going to a new minor floor).
- Before bumping: skim the zod changelog between the currently-locked version and the
  target for anything touching `.safeParse`/error formatting, since those are the most
  commonly relied-on APIs in this repo's validators.

## .nvmrc (issue #1196)

- No `.nvmrc` exists at the repo root today.
- [.github/workflows/cli-checks.yml](../.github/workflows/cli-checks.yml) pins CI to
  `node-version: 20` via `actions/setup-node`. That's the version an `.nvmrc` should
  match, so `nvm use` locally lines up with what CI actually runs.
- [CONTRIBUTING.md](../CONTRIBUTING.md) currently documents the prerequisite loosely as
  "Node.js 18+ or 20+" — once an `.nvmrc` pins a single version, that line should be
  tightened to match rather than left as a range.

## Pin Node in the deploy script (issue #1195)

- The actual gap is in
  [.github/workflows/deploy-mainnet.yml](../.github/workflows/deploy-mainnet.yml): the
  `simulate-contracts` and `deploy-contracts` jobs both invoke `node scripts/deploy.js`
  directly (`--simulate-only` and full run, respectively) with **no `actions/setup-node`
  step at all** — Node comes from whatever `ubuntu-latest` ships with that day. The
  `deploy-frontend` job in the same workflow, by contrast, already pins
  `node-version: '20'` via `actions/setup-node@v6`. So the fix is adding an equivalent
  `actions/setup-node` step (pinned to Node 20, matching `deploy-frontend` and
  `cli-checks.yml`) to `simulate-contracts` and `deploy-contracts`.
- Related drift spotted while looking at this: `.github/workflows/nightly-tier-upgrade.yml`
  pins `node-version: '22'`, out of step with every other workflow in the repo (20). Not
  in scope for #1195, but worth a separate ticket — it means the "one Node version"
  story isn't fully true yet even after #1195/#1196 land.
