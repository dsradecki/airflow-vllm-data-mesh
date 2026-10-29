# data-mesh-airflow-vllm

## Spec
@openspec/specs/batch-inference/proposal.md
@openspec/specs/batch-inference/spec.md
@openspec/specs/batch-inference/design.md

The spec is the agreed contract between tenants and MLOps (source of truth). The
proposal holds the design problems (P1–P4) and open behavioural questions. design.md
records the decided mechanism for each design problem.

## Changes: one per design problem
- Each design problem in the proposal's table is one change
  `openspec/changes/p<N>-<topic>/` with proposal.md, design.md and tasks.md. It
  decides the problem's design and implements every requirement the table lists for it.
- Work the problems in the order of the table.
- Spec changes go into the problem's change as deltas
  (`## ADDED / MODIFIED / REMOVED Requirements`). Never edit `openspec/specs/` directly.
- When every task in a change is ticked, write the decision into that problem's section
  in `openspec/specs/batch-inference/design.md` and move the change to
  `openspec/changes/archive/YYYY-MM-DD-p<N>-<topic>/`.

## Session protocol: one task per session
- Each session does exactly one task from a problem change's tasks.md: either the
  design decision or one `### Requirement:`. If I haven't named one, ask. Don't pick
  one yourself.
- A problem's requirements are implemented only after its design task is ticked.
- The requirement's scenarios are the acceptance criteria. Each scenario needs a test.
  If the change has a delta for the requirement, the delta wording applies.
- Don't implement other requirements, even closely related ones. If this one depends on
  something that isn't built yet, don't propose or build stubs for it. Name the
  dependency and ask me how to proceed.
- The spec describes behaviour only. Mechanism choices (queue tech, storage, etc.) go
  in the change's design.md.
- Never edit the spec or a proposal silently. If the spec looks wrong or ambiguous,
  stop and ask.
- Before finishing: tests pass, the task is ticked in the change's tasks.md with notes
  for later sessions, and there's one commit: `feat(batch-inference): <Requirement name>`
  for a requirement, `docs(batch-inference): P<N> design` for a design decision.
