---
description: Start a session on one spec requirement
argument-hint: <Requirement name>
---
This session implements the requirement "$ARGUMENTS" from openspec/specs/batch-inference/spec.md.

1. Quote that requirement and its scenarios exactly. If the owning change has a delta for
   it, quote the delta wording. If no heading matches, list the closest ones and stop.
2. Find the problem that covers it in the proposal's table and its change in
   openspec/changes/p<N>-*/. If that change's design task isn't ticked, say so and stop.
   Otherwise read the change's design.md and the notes in its tasks.md.
3. Propose a short plan: files to touch, one test per scenario, and any dependencies on
   things not built yet (don't propose stubs for them). Wait for my OK before writing code.
