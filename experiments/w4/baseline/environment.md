# W4 Baseline Environment Preparation

## Repository identity

- Date: 2026-10-09 (Pacific/Auckland)
- Baseline: `team/main`
- Baseline commit: `5a2f009e2f6d266087f2f3f527e6e384cefd6de4`
- Experiment branch: `codex/experiment/w4-multi-agent-rewrite`
- Node: `v24.14.0` (`.nvmrc` specifies Node 24)
- pnpm: `11.2.1` (matches `package.json`)

## Dependency preparation result

Status: FAILED - environment not ready for baseline verification.

Observed failures:

1. Native `better-sqlite3@13.0.3` installation attempted a local `node-gyp`
   build. Visual Studio Build Tools were detected, but no Windows SDK was
   available, so the native build failed.
2. Repeated pnpm installs, including a clean install with
   `--frozen-lockfile --ignore-scripts`, failed near completion with random
   `ERR_PNPM_ENOENT` errors while importing packages into hoisted
   `node_modules` temporary directories.

Consequences:

- Build, typecheck, lint, format, and tests have not yet been established as
  passing at this baseline.
- Database-dependent tests must not be reported as passing.
- Environment preparation time is separate from W4 agent execution time.
- The initial evidence-only commit may bypass the pre-commit hook because the
  hook invokes `pnpm preflight`, which cannot run until dependency preparation
  is repaired. This bypass does not count as a successful preflight result.
