# W4 Architecture Decision Log

Record decisions only after repository evidence and human review. Do not use
this document for unverified agent assumptions.

| ID | Date | Status | Decision | Evidence | Consequences | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| W4-000 | 2026-10-09 | Accepted | Use `team/main` commit `5a2f009e2f6d266087f2f3f527e6e384cefd6de4` as the W4 experiment baseline. | Git branch preparation record | Later changes to `team/main` are not silently incorporated into experimental comparisons. | Human coordinator |

## Unresolved questions

| ID | Question | Why it matters | Required evidence | Owner |
| --- | --- | --- | --- | --- |
| W4-Q001 | What is the first migration vertical slice? | It determines the first shared contract and agent overlap. | Dependency map and baseline test coverage | Unassigned |
| W4-Q002 | How will existing NextAuth sessions remain valid when Python becomes the authentication authority? | Session invalidation would violate compatibility rules. | Current cookie/JWT implementation and compatibility experiment | Unassigned |
| W4-Q003 | Which queue backend and payload compatibility path will Python workers use? | Retry, recovery, scheduling, and idempotency must be preserved. | Queue plugin and worker contract inventory | Unassigned |
