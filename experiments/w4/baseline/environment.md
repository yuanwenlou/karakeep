# W4 Baseline Environment Preparation

## Repository identity

- Date: 2026-10-09 to 2026-10-10 (Pacific/Auckland)
- Baseline branch: `team/main`
- Baseline source commit: `5a2f009e2f6d266087f2f3f527e6e384cefd6de4`
- Experiment preparation commit: `72d378725d3ab4cfcb1b30cfe15b0963540e5f60`
- Experiment branch: `codex/experiment/w4-multi-agent-rewrite`
- Personal fork remote: `origin` (`yuanwenlou/karakeep`)
- Team remote: `team` (`tanjuntao-dev/karakeep`)
- Official upstream remote: `upstream` (`karakeep-app/karakeep`)
- Node: `v24.14.0` (`.nvmrc` specifies Node 24)
- pnpm: `11.2.1` (matches `package.json`)

The preparation commit adds experiment evidence files only. Product source is
unchanged from the baseline source commit.

## Dependency preparation

Final status: **READY for focused development and verification**.

Initial attempts failed for two independent environment reasons:

1. A normal install attempted a local `better-sqlite3@13.0.3` node-gyp build.
   Visual Studio Build Tools were detected, but the installation was incomplete
   and did not expose a Windows SDK to node-gyp.
2. Repeated hoisted installs failed near completion with intermittent
   `ERR_PNPM_ENOENT` errors in `node_modules/*_tmp_*` directories while IDEA was
   indexing the project.

The successful preparation used a clean `node_modules`, closed IDEA during the
install, limited pnpm package-import workers, used the frozen lockfile, and
skipped lifecycle scripts. The installed `better-sqlite3` package contains a
Windows x64 prebuild. Both `better-sqlite3/win32-x64` and the package's normal
default entry point successfully created an in-memory database and executed
`select 1`.

Repository-local Git configuration was changed from `core.autocrlf=true` to
`core.autocrlf=input`. The repository formatter left no Git-visible
product-source diff. One generated artifact,
`packages/open-api/karakeep-openapi-spec.json`, nevertheless remains CRLF in the
Windows working copy; Git's line-ending normalization hides that byte-level
difference from `git diff`.

## Verification results

### Passing checks

| Command/check | Result |
| --- | --- |
| `pnpm typecheck` | PASS: 23/23 tasks |
| `pnpm lint` | PASS: 20/20 tasks, zero reported warnings or errors |
| `pnpm preflight` | PASS: 64/64 typecheck, lint, and format tasks |
| `better-sqlite3` runtime smoke test | PASS: default and Windows prebuild entry points returned `1` from an in-memory query |
| Targeted tags baseline test | PASS: 1 file, 30/30 tests |

### Known baseline limitations

| Command/check | Result | Classification |
| --- | --- | --- |
| `pnpm build` | FAIL: Next.js aborted while renaming a config file in `%APPDATA%` with Windows `EXDEV`; Turbo stopped the remaining build tasks | Environment-specific; full build is not established as passing |
| `CI=1 pnpm test` | FAIL: completed output contained at least 238 passing and 3 failing tests; unfinished tRPC, plugin, and e2e tasks were terminated after the web failure | Partial baseline only; not a full-suite count |
| Web tests at local timezone | 28 passed, 3 failed in the web package before Turbo stopped | Failures depend on timezone/default locale expectations |
| Web tests with `TZ=UTC` | 25 passed, 2 assertion failures, plus one suite collection failure caused by injected `NO_COLOR` | Remaining assertions depend on Windows default locale; collection failure is environment-variable validation |
| Full tRPC test command with injected `NO_COLOR` | 2 files collected and 5 tests passed; 19 suites failed collection before assertions | Environment-variable validation failure, not 19 behavioural test failures |
| `pnpm --filter @karakeep/open-api check` | FAIL with no semantic or Git-visible specification diff; the checked-in working-copy file contains 5,436 CRLF line endings while the generator emits LF | Windows line-ending mismatch in a raw string comparison |

The executor-provided `NO_COLOR` value is rejected by the project's Zod
environment schema. After removing `NO_COLOR` for the targeted command,
`packages/trpc/routers/tags.test.ts` passed all 30 tests.

The OpenAPI check reads the existing JSON and compares it byte-for-byte with
`JSON.stringify` output. A separately generated specification was structurally
identical and produced no `git diff --no-index`; the command still exited 1
because the working-copy file uses CRLF. This pre-existing platform issue is why
the documentation-only preparation commit may need to bypass the Git hook. It
must not be reported as a successful OpenAPI check.

## Tooling warning

Turbo 2.5.5 reports that it cannot fully parse the current
`patchedDependencies` lockfile representation. It still ran the workspace
checks and reported all 64 preflight tasks successful. This warning is retained
as baseline evidence and is not being fixed by changing project dependencies,
because that would alter the shared experiment baseline.

## Baseline interpretation

- Dependency installation and the standard static checks are usable.
- The full build and full test suite are not green in this Windows environment.
- Focused migration work must use targeted tests with explicit environment
  controls and must not claim the uncompleted root test suite passed.
- Environment preparation time is separate from Gemini W4 execution time.
