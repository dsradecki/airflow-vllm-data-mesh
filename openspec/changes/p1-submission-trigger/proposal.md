# Change: P1 — submission trigger

## Why

P1 (trigger between tenants and MLOps internal Airflow, listed in
`specs/batch-inference/proposal.md`) is open. The Job Submission
requirement currently names the API Gateway, which is a mechanism, not a
behaviour. Options under discussion include submitting without the gateway
(Pub/Sub only), so the requirement should stay mechanism-neutral and let
`design.md` carry the choice.

## What Changes

- Decides the P1 design (`design.md`). Status: open.
- Implements the requirements P1 covers: Job Submission, Job Lifecycle and
  Status, Idempotent Submission, Durable Job State, Cancellation, Tenant
  Isolation.
- **MODIFIED** `batch-inference` / Job Submission: "through the API Gateway"
  becomes "through the platform's submission interface". The scenario is
  unchanged.

## Impact

- Specs: `batch-inference` (Job Submission)
- `specs/batch-inference/proposal.md` still lists "API Gateway integration"
  under Impact. Left unchanged until P1 is decided.
