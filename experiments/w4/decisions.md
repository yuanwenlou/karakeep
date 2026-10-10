# W4 Architecture Decision Log

Record decisions only after repository evidence and human review. Do not use
this document for unverified agent assumptions.

| ID | Date | Status | Decision | Evidence | Consequences | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| W4-000 | 2026-10-09 | Accepted | Use `team/main` commit `5a2f009e2f6d266087f2f3f527e6e384cefd6de4` as the W4 experiment baseline. | Git branch preparation record | Later changes to `team/main` are not silently incorporated into experimental comparisons. | Human coordinator |
| W4-001 | 2026-10-10 | Accepted | Use the existing six Tags REST operations as the first Python vertical slice; do not add REST `merge` or `delete-unused` in this slice. | S01 architecture analysis and human review | The result is honestly classified as a partial Tags migration; existing tRPC-only operations remain TypeScript responsibilities. | Human coordinator |
| W4-002 | 2026-10-10 | Accepted | Build the Python slice under `backend/` with FastAPI and SQLAlchemy 2.0 while preserving the existing SQLite schema. | S01 comparison of candidate slices | The Python service is additive during the experiment and must not alter existing database migrations. | Human coordinator |
| W4-003 | 2026-10-10 | Accepted | Limit the first slice to API-key authentication; NextAuth cookie compatibility remains out of scope for this slice. | S01 client-flow analysis | MCP/SDK can be used for live REST verification while existing browser clients continue using TypeScript tRPC. | Human coordinator |
| W4-004 | 2026-10-10 | Accepted | Execute B1 persistence first, measure quota, then continue B2 API/Auth, B3 queue compatibility, and B4 verification only after review. | S01 consumed 30.03 percentage points of weekly Gemini quota | Agent contexts and file ownership must be narrow; quota-reset waiting time is not counted as agent execution time. | Human coordinator |

## Unresolved questions

| ID | Question | Why it matters | Required evidence | Owner |
| --- | --- | --- | --- | --- |
| W4-Q002 | How will existing NextAuth sessions remain valid when Python becomes the authentication authority? | Session invalidation would violate compatibility rules. | Current cookie/JWT implementation and compatibility experiment | Unassigned |
| W4-Q003 | Which queue backend and payload compatibility path will Python workers use? | Retry, recovery, scheduling, and idempotency must be preserved. | Queue plugin and worker contract inventory | Unassigned |
