# Design: p1-submission-trigger

## P1: Trigger between tenants and MLOps internal Airflow

Status: **open**. Options under discussion, nothing decided.

### Option A: HTTP via internal API Gateway
- Tenant calls the gateway over HTTP; the gateway puts a message on a queue.
- Synchronous validation (e.g. unregistered model rejected immediately).
- Tenant identity from gateway auth.
- Tenants depend only on an HTTP contract.

### Option B: Pub/Sub only
- Tenant publishes a submission message to a platform-owned topic.
- Job ID: the Pub/Sub message ID (with per-tenant topics: topic + message ID).
- Job is `QUEUED` once the publish succeeds (durably stored by Pub/Sub).
- Tenant identity: one topic per tenant, IAM-restricted publish. Message
  attributes set by the tenant can't be trusted for identity.
- Validation is asynchronous: an invalid submission gets an ID, then fails
  with a tenant reason category.
- Idempotency: publisher retries can duplicate; the consumer dedupes on
  (tenant, idempotency key). Belongs to Idempotent Submission.
- Tenant side: the Airflow Google provider's Pub/Sub publish operator.

### Completion signal (relates to Job Lifecycle and Status)
- Event-driven, not polling: a completion topic tenants subscribe to,
  filtered by tenant.
- Tenant side: a deferrable operator that waits on the completion event
  without holding a worker slot.
- Mechanism not decided.

### Open questions
- A or B?
- If B: is asynchronous rejection of invalid submissions acceptable to tenants?
- Queue between submission intake and Airflow: not decided, out of scope
  for Job Submission.
