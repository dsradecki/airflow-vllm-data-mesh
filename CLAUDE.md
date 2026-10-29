# data-mesh-airflow-vllm

## Active change
@openspec/changes/add-batch-inference-data-mesh/proposal.md
@openspec/changes/add-batch-inference-data-mesh/specs/batch-inference/spec.md
@openspec/changes/add-batch-inference-data-mesh/tasks.md

## Session protocol: one requirement per session
- Each session implements exactly one `### Requirement:` from spec.md. If I haven't
  named one, ask. Don't pick one yourself.
- That requirement's scenarios are the acceptance criteria. Each scenario needs a test.
- Don't implement other requirements, even closely related ones. If this one depends on
  something that isn't built yet, use a minimal stub and record it under that
  requirement in tasks.md.
- spec.md describes behaviour only. Record mechanism choices (queue tech, storage, etc.)
  in the change's design.md, under the design problem (P1–P4 in proposal.md) they belong to.
- Never edit spec.md or proposal.md silently. If the spec looks wrong or ambiguous,
  stop and ask.
- Before finishing: tests pass, the requirement is ticked in tasks.md with notes for
  later sessions, and there's one commit `feat(batch-inference): <Requirement name>`.
