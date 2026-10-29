## ADDED Requirements

### Requirement: Job Submission

The system SHALL let a tenant submit a batch inference job through the API
Gateway with exactly two tenant-controlled inputs: `model_name` and
`data_path`. When a submission is valid, the system SHALL accept it and return
a job identifier that is unique across all tenants.

#### Scenario: Valid submission

- **WHEN** a tenant submits a job with a registered `model_name` and a `data_path`
- **THEN** the system accepts the job
- **AND** returns a unique job identifier
- **AND** the job is in state `QUEUED` or `RUNNING`

### Requirement: Operator-Managed Execution Parameters

The system SHALL keep execution parameters (such as batch size, chunk size,
per-task concurrency, and timeouts) under MLOps control, configurable per
model. Tenants MUST NOT be able to set or override them.

### Requirement: Load-Independent Admission

The system MUST accept every valid submission regardless of current load.
Reasons it MUST NOT use to reject a submission include: maximum active runs
reached, all workers busy, model at capacity, or queue non-empty.

#### Scenario: Model saturated

- **WHEN** a tenant submits a valid job for a model whose batch capacity is fully in use
- **THEN** the system accepts the job and returns a job identifier
- **AND** the job is in state `QUEUED`
- **AND** the response carries no error

#### Scenario: Burst of submissions

- **WHEN** many tenants submit valid jobs for the same model within a short window
- **THEN** every submission is accepted
- **AND** every job is queued for execution

### Requirement: Idempotent Submission

The system SHALL let a tenant attach an idempotency key to a submission.
Repeating a submission with the same tenant and idempotency key MUST NOT create
a second job.

#### Scenario: Tenant retries after a network error

- **WHEN** a tenant re-sends a submission with an idempotency key it already used
- **THEN** the system returns the job identifier of the original job
- **AND** no new job is created

#### Scenario: Same key used by different tenants

- **WHEN** two different tenants submit with the same idempotency key
- **THEN** the system treats them as two independent jobs

### Requirement: Per-Model Queuing

Jobs that target the same model and can't start right away SHALL be queued.
Every accepted job SHALL eventually run unless it is cancelled or the model is
withdrawn.

#### Scenario: Job waits for capacity

- **WHEN** a job is queued because its model has no free batch capacity
- **AND** capacity for that model becomes available
- **THEN** a queued job for that model moves to `RUNNING` without the tenant doing anything

#### Scenario: No silent drops

- **WHEN** a job has been accepted
- **THEN** it reaches exactly one terminal state (`SUCCEEDED`, `FAILED`, or `CANCELLED`)
- **AND** it is never lost from the system's records

### Requirement: Per-Model Isolation

A backlog or failure for one model MUST NOT delay or fail jobs for other
models.

#### Scenario: Backlog on one model

- **WHEN** model A has a deep queue of jobs
- **AND** a tenant submits a job for model B, which has free capacity
- **THEN** the job for model B starts without waiting for model A's queue

#### Scenario: Endpoint outage on one model

- **WHEN** the vLLM endpoint for model A is unavailable
- **THEN** jobs for other models keep running normally

### Requirement: Fairness Between Tenants

When multiple tenants have jobs queued for the same model, one tenant MUST NOT
be able to keep other tenants' jobs from starting indefinitely.

#### Scenario: One tenant floods the queue

- **WHEN** tenant X has queued many jobs for a model
- **AND** tenant Y then queues one job for the same model
- **THEN** tenant Y's job starts within a bounded time that doesn't grow without limit with the number of tenant X's jobs

### Requirement: Durable Job State

Job state SHALL survive restarts, redeployments, and node loss of platform
components.

#### Scenario: Orchestrator restart while jobs are queued

- **WHEN** platform components (scheduler, workers, or the trigger path) restart while jobs are `QUEUED`
- **THEN** those jobs stay queued after the restart and are executed later

#### Scenario: Worker loss while a job is running

- **WHEN** a worker running part of a job is lost (e.g. node preemption or pod eviction)
- **THEN** the job keeps going or resumes without tenant action
- **AND** it doesn't end in `FAILED` just because the worker was lost

### Requirement: Job Lifecycle and Status

The system SHALL expose each job's state to the tenant that owns it. States
are `QUEUED`, `RUNNING`, `SUCCEEDED`, `FAILED`, and `CANCELLED`. Status SHALL
include progress while running, the output location on success, and a reason
category on failure (tenant input vs. platform).

#### Scenario: Tenant polls a queued job

- **WHEN** a tenant queries the status of a job in `QUEUED`
- **THEN** the response includes the state and the time the job was accepted

#### Scenario: Tenant polls a running job

- **WHEN** a tenant queries the status of a job in `RUNNING`
- **THEN** the response includes progress (e.g. requests processed out of total)

#### Scenario: Job completes

- **WHEN** a job finishes successfully
- **THEN** its state becomes `SUCCEEDED`
- **AND** the status includes the output location

#### Scenario: Tenant waits for completion without polling

- **WHEN** a job reaches a terminal state
- **THEN** the system emits a completion signal that the tenant's automation can consume

### Requirement: Transient Fault Absorption

Platform-side transient conditions MUST NOT put a job in `FAILED`. These
include: busy workers, vLLM replica restarts or scale events, temporary
endpoint unavailability, and worker preemption. The system SHALL put a job in
`FAILED` only when the cause is tenant-attributable or when a platform fault
lasts beyond an MLOps-defined limit.

#### Scenario: Temporary vLLM unavailability

- **WHEN** the vLLM endpoint for a job's model is briefly unavailable during execution
- **THEN** the job stays `RUNNING` and continues once the endpoint recovers

#### Scenario: Prolonged platform fault

- **WHEN** a platform fault lasts longer than the MLOps-defined limit
- **THEN** the job ends in `FAILED` with a platform reason category
- **AND** MLOps is alerted

### Requirement: Cancellation

A tenant SHALL be able to cancel a job it owns that isn't yet terminal.

#### Scenario: Cancel a queued job

- **WHEN** a tenant cancels a job in `QUEUED`
- **THEN** the job moves to `CANCELLED` and never starts

#### Scenario: Cancel a running job

- **WHEN** a tenant cancels a job in `RUNNING`
- **THEN** the system stops sending that job's requests to vLLM within a bounded time
- **AND** the job moves to `CANCELLED`

### Requirement: Tenant Isolation

A tenant SHALL be able to see, query, and cancel only its own jobs. Every job
and its resource usage SHALL be attributed to the tenant that submitted it.

#### Scenario: Access to another tenant's job

- **WHEN** tenant X queries or cancels a job owned by tenant Y
- **THEN** the system refuses the request without revealing anything about the job

### Requirement: Wait-Time Visibility

The system SHALL tell tenants how long their queued jobs are likely to wait,
so tenant automation can set realistic timeouts.

#### Scenario: Estimated wait on submission or status

- **WHEN** a job is `QUEUED`
- **THEN** its status includes its queue position or an estimated start time for its model (estimated time might be hard, queue position at least)

### Requirement: Bounded Queue Wait Time

Under load within the platform's maximum capacity, queue wait time per model
SHALL stay within an MLOps-configured target.

#### Scenario: Demand rises within maximum capacity

- **WHEN** queued demand for a model grows
- **AND** the platform hasn't reached its maximum capacity for that model
- **THEN** queue wait time for that model stays within the configured target

#### Scenario: Demand exceeds maximum capacity

- **WHEN** queued demand for a model exceeds what maximum capacity can serve within the target
- **THEN** jobs are still accepted and stay queued (they are not rejected)
- **AND** MLOps is alerted that the wait-time target is being breached
- **AND** tenants see the longer estimated wait in job status

### Requirement: Capacity Scaling

The system SHALL change the batch processing capacity for a model in response
to that model's queued and running batch demand, within MLOps-defined minimum
and maximum bounds.

#### Scenario: Scale up on backlog

- **WHEN** queued batch demand for a model stays above what current capacity can serve within the wait-time target
- **THEN** the system increases capacity for that model, up to the configured maximum

#### Scenario: Scale down when idle

- **WHEN** batch demand for a model drops
- **THEN** the system reduces capacity for that model, down to the configured minimum for online traffic
- **AND** in-flight requests aren't failed by the scale-down

### Requirement: Online Traffic Protection

Because the vLLM endpoints also serve online traffic, batch load MUST NOT push
online traffic outside its service level objective. When the two compete,
online traffic SHALL take precedence, and batch jobs SHALL slow down instead of
failing.

#### Scenario: Online spike during batch execution

- **WHEN** online traffic to a model rises sharply while batch jobs for that model are running
- **THEN** online latency stays within its SLO
- **AND** batch throughput for that model may drop
- **AND** running batch jobs don't move to `FAILED` because of the reduced throughput
