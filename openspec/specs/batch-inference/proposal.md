# Change: Batch inference as a data mesh product

## Why

Tenants (other teams, mostly running their own Airflow DAGs) need to run large
JSONL batches against the vLLM models MLOps already serves. Today the batching
pipeline exists (Airflow on GKE with Celery workers; tasks process chunks of
requests with thread-pool backpressure against vLLM endpoints), but nothing
defines how tenants use it as a shared, self-service product.

In data mesh terms, batch inference is a self-serve data product with a clear
split of ownership. Tenants own the **what**: which model to call and which
data to run it on. MLOps owns the **how**: orchestration, batching parameters,
concurrency, capacity, and operating the vLLM endpoints. The contract between
them is the job interface and the guarantees in this spec. Tenants never need
to know how jobs are executed, and MLOps can change execution without breaking
tenants.

Tenants are automated systems such as scheduled Airflow DAGs. They can't handle "busy, try later" responses,
so the platform has to absorb contention instead of handing it to them.

## What Changes

- **ADDED** capability `batch-inference`: the tenant-visible and operator-visible
  behaviour of the batch inference product:
  - Submitting a job with only `model_name` and `data_path`
  - Execution parameters (batch size, concurrency, etc.) owned by MLOps
  - Admission that never rejects a valid job because of load
  - Per-model queuing, fairness between tenants, and durability
  - Job lifecycle, status, completion signal, and cancellation
  - Telling transient platform faults apart from terminal failures
  - Bounded wait times under load, plus capacity scaling
  - Protecting online traffic that shares the same vLLM endpoints

## Scope

This change defines **behaviour only**. It does not choose mechanisms.

In scope:

- What tenants can submit, observe, and rely on
- What MLOps can configure
- Guarantees about queuing, failure, and wait time

Out of scope (already solved or handled elsewhere):

- Batching internals inside a task: chunking, thread-pool backpressure,
  request-level retries against vLLM
- vLLM serving configuration

## Follow-up Changes

Each design problem below becomes one change under
`openspec/changes/p<N>-<topic>/`. That change decides the problem's design and
implements every requirement listed for it. Each requirement belongs to exactly
one problem.

## Open Design Problems (to be resolved one by one in later changes)

Problems are worked in the order of this table. Whatever we choose has to
satisfy the requirements in `specs/batch-inference/spec.md`.

| # | Problem | Requirements it covers |
|---|---------|------------------------|
| P1 | Trigger between tenants and MLOps internal Airflow | Job Submission, Job Lifecycle and Status, Idempotent Submission, Durable Job State, Cancellation, Tenant Isolation |
| P3 | Multiple tenants targeting the same model at once | Load-Independent Admission, Per-Model Queuing, Fairness Between Tenants, Transient Fault Absorption |
| P2 | DAG topology (e.g. one DAG per vLLM endpoint or not) | Operator-Managed Execution Parameters, Per-Model Isolation |
| P4 | Autoscaling when demand for one model exceeds capacity | Bounded Queue Wait Time, Capacity Scaling, Online Traffic Protection, Wait-Time Visibility |

## Open Behavioural Questions

These need answers before or during P1–P4:

- **Ordering policy:** strict FIFO per model, or fair-share across tenants
  (e.g. round-robin)?
- **Priorities:** do some tenants or jobs get priority classes?
- **Output location:** does the tenant supply `output_path`, or does the
  platform pick it and report it?
- **Model removal:** what happens to queued jobs when a model is retired?
- **Wait-time target:** what is the target queue wait per model, and does a
  tenant get to declare a per-job deadline?
- **Online traffic SLO:** what latency or throughput budget for online traffic
  must batch load respect?
- **Quotas:** are there per-tenant limits on queued jobs or total requests?
  If so, what happens when a quota is exceeded, given that capacity rejections
  are disallowed?

## Impact

- New spec: `batch-inference`
- Affected systems: API Gateway integration, MLOps Airflow instance, vLLM
  deployments on GKE, tenant DAGs (as consumers)
