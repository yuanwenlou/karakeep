# W4 Multi-Agent Collaborative Rewrite

## Experiment identity

- Workflow: W4 - Multi-Agent Collaborative Rewrite
- Rewrite target: Karakeep TypeScript backend to Python
- Baseline remote and branch: `team/main`
- Baseline commit: `5a2f009e2f6d266087f2f3f527e6e384cefd6de4`
- Experiment branch: `codex/experiment/w4-multi-agent-rewrite`
- Primary execution agent: Gemini
- Human role: coordinator, reviewer, and integration decision-maker

## Research question

Can multiple specialised coding agents improve the overall efficiency of a
large-scale backend rewrite without introducing excessive coordination,
integration, architectural-consistency, or human-review costs?

## Ground rules

- Preserve user-facing behaviour, database compatibility, validation,
  authorization, error semantics, worker contracts, and configuration.
- Do not add unrelated features or redesign infrastructure for convenience.
- Do not weaken, delete, or disable tests to conceal failures.
- Do not claim that unexecuted tests passed.
- Each session must have an explicit role, file ownership, inputs, outputs,
  and verification expectations.
- Concurrent agents should use separate branches or worktrees and avoid
  editing the same files.
- Human review is required before integration.
- Do not open a pull request against the upstream Karakeep repository.

## Status vocabulary

- Fully Migrated
- Partially Migrated
- Implemented but Not Verified
- Not Migrated

## Evidence layout

- `evidence-log.csv`: one row per significant session.
- `sessions/`: prompts, transcripts, command output, and reviews.
- `decisions.md`: architectural decisions and unresolved questions.
- `migration-status.md`: component implementation and verification status.
- `templates/session-record.md`: required session record format.

## Baseline policy

Record baseline commands and results before rewrite implementation starts.
Mark unexecuted commands as `NOT RUN` and record failures honestly.
