# W4 Repository-Level Backend Investigation

## Purpose and boundary

This is pre-experiment coordination evidence prepared by Codex. It records
confirmed repository facts needed to design Gemini's W4 multi-agent sessions.
No product source was modified during this investigation.

## Confirmed request path and backend layers

The current server request path is:

```text
TypeScript clients / Next.js SSR
        |
        v
apps/web/app/api/[[...route]]/route.ts
        |
        v
packages/api (Hono REST routes and tRPC adapter)
        |
        v
packages/trpc (authorization, validation, business logic, models)
        |
        +--> packages/db (Drizzle ORM and SQLite schema)
        +--> packages/shared-server (queues, plugins, event logging, assets)
        +--> packages/shared (configuration, schemas, shared utilities)

apps/workers also imports packages/db, packages/shared-server,
packages/shared, and selected packages/trpc models and business utilities.
```

Authentication context is currently constructed in the Next.js application
and injected into Hono. REST handlers commonly call an in-process tRPC caller,
so the REST layer is not yet an independent business-logic implementation.

## Measured scope

Counts are approximate static measurements from tracked TypeScript files at the
experiment preparation commit.

| Area | TS/TSX files | Lines | Test files | Approx. tests |
| --- | ---: | ---: | ---: | ---: |
| `packages/trpc` | 74 | 26,030 | 21 | 79 by simple static count |
| `packages/api` | 32 | 1,901 | 3 | 8 |
| `packages/db` | 7 | 1,413 | 0 | 0 |
| `packages/shared-server` | 12 | 1,497 | 1 | 8 |
| `apps/workers` | 49 | 10,534 | 5 | 27 |
| `packages/shared` | 68 | 8,519 | 11 | 110 |
| `packages/open-api` | 15 | 2,493 | 0 | 0 |

The tRPC router exposes approximately 146 query/mutation procedures across 18
router modules. The existing Hono REST route files contain approximately 75
HTTP method handlers. These are inventory counts, not migration-completion
units.

## Major migration boundaries

### Python API boundary

- Replace Hono/tRPC-backed server behaviour with Python REST endpoints.
- Generate OpenAPI from the Python implementation.
- Preserve current REST paths, status codes, authentication, pagination,
  validation, and response shapes.
- Adapt TypeScript clients where they still call tRPC directly.

### Python persistence boundary

- Map the existing SQLite schema to SQLAlchemy without redesigning it.
- Preserve table/column names, relationships, indexes, constraints, and
  transaction behaviour.
- Treat the existing Drizzle migrations and live schema as compatibility
  inputs, not permission to create a new schema.

### Authentication boundary

- Python must eventually become authoritative for sessions, API keys,
  authorization, and scopes.
- Current session context originates in Next.js, so session compatibility is a
  sequencing constraint rather than an isolated module conversion.

### Worker boundary

- Workers directly import database tables, queues, shared server services, and
  selected tRPC models.
- Worker migration cannot safely proceed independently of the Python database,
  queue contracts, and business-service boundaries.

## High-coupling and high-risk areas

1. **Authentication and sessions:** spans NextAuth, request-context creation,
   API keys, scopes, tRPC procedures, REST middleware, and clients.
2. **Bookmarks:** central model with search, vector search, assets, tags, lists,
   queues, AI, crawling, and worker side effects.
3. **Workers:** directly reuse TypeScript backend internals rather than a stable
   service boundary.
4. **Database compatibility:** `packages/db` has no package-local tests, so
   compatibility is verified indirectly through higher-level business tests.
5. **Configuration:** modules parse environment variables at import time;
   executor-provided values such as `NO_COLOR` can prevent test collection.
6. **REST/tRPC duplication:** REST routes often delegate to tRPC callers. A
   Python endpoint can appear complete while still depending on TypeScript
   authority unless dependency replacement is checked explicitly.
7. **Client spread:** web, mobile, browser extension, CLI, MCP, shared React
   hooks, and SDK do not all use the same communication path.

## Recommended first vertical slice

**Candidate: Tags REST/API slice. This is proposed, not yet an accepted
architecture decision.**

Confirmed characteristics:

- 7 tRPC procedures in `packages/trpc/routers/tags.ts`.
- 6 existing REST endpoints in `packages/api/routes/tags.ts`.
- Existing OpenAPI definitions in `packages/open-api/lib/tags.ts`.
- SQL persistence through `bookmarkTags` and `tagsOnBookmarks`.
- User ownership, API-key scope checks, validation, conflict handling,
  pagination, filtering, sorting, transactions, and search-reindex side effects.
- Existing targeted behavioural baseline: 30/30 tests passing after removing
  the executor's invalid `NO_COLOR` value.
- Multiple consumers exist across web, mobile, browser extension, CLI,
  shared-react, and SDK, making client compatibility observable.

Why it is preferable to other starting points:

- Health/version endpoints are safer but too shallow to evaluate a meaningful
  backend rewrite workflow.
- Authentication, bookmarks, and workers are too highly coupled for the first
  controlled experiment.
- Tags exercises the intended architecture while retaining a bounded contract
  and strong existing tests.

Important risks:

- Tag merge/delete trigger bookmark search reindexing.
- Some tag operations exist in tRPC but are not exposed by the current REST
  file, so scope must distinguish REST parity from full tRPC feature parity.
- Client migration cannot be claimed complete until all relevant tRPC callers
  are inventoried and adapted or explicitly left transitional.

## Proposed controlled W4 decomposition

### Analysis wave - may run in parallel

1. **Architecture agent:** verify layer boundaries and produce a dependency
   graph; read-only.
2. **API/client contract agent:** inventory tag endpoints, schemas, errors, and
   every client call site; read-only.
3. **Database compatibility agent:** map tag-related Drizzle schema,
   constraints, queries, transactions, and side effects to SQLAlchemy
   requirements; read-only.
4. **Verification agent:** map existing tests to compatibility assertions and
   design Python contract/integration tests; read-only.

These agents may run concurrently because they own separate evidence outputs
and do not edit product code.

### Synthesis gate - sequential

Human/Codex review must reconcile the four reports, resolve inconsistent
assumptions, accept the initial architecture, and freeze shared contracts before
implementation begins.

### Implementation wave - partly sequential

1. Python service foundation and agreed project structure must be implemented
   first because later agents depend on its interfaces.
2. SQLAlchemy tag models/repository and tag REST service may then be assigned
   with non-overlapping ownership if their interfaces are frozen.
3. Client adaptation should begin only after the REST contract is executable.
4. Verification/review remains independent and must not weaken the original
   TypeScript assertions.

## Shared context required for every Gemini agent

- `AGENTS.md`
- `experiments/w4/README.md`
- the shared General Rules document
- this repository investigation
- the agent's exact role and writable file ownership
- start commit and required output path
- explicit acceptance checks
- instruction not to edit or disable unrelated tests
- instruction to label confirmed facts, proposals, assumptions, and unresolved
  questions separately
- instruction not to claim unexecuted checks passed

## Unresolved questions before implementation

1. Exact Python framework and repository layout.
2. Whether the first slice targets existing REST parity only or every tag tRPC
   operation, including merge and delete-unused.
3. How Python receives and validates the current session during coexistence.
4. How search-reindex side effects cross the temporary TypeScript/Python
   boundary.
5. Which client is adapted first for end-to-end verification.
