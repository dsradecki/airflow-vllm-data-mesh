# Design: batch-inference

Decided mechanisms for the design problems in proposal.md, in the order they
are worked. Each section is filled in once its `p<N>` change is decided.

## P1: Trigger between tenants and MLOps internal Airflow

Covers: Job Submission, Job Lifecycle and Status, Idempotent Submission,
Durable Job State, Cancellation, Tenant Isolation.

Status: open (`changes/p1-submission-trigger`).

## P3: Multiple tenants targeting the same model at once

Covers: Load-Independent Admission, Per-Model Queuing, Fairness Between
Tenants, Transient Fault Absorption.

Status: open.

## P2: DAG topology

Covers: Operator-Managed Execution Parameters, Per-Model Isolation.

Status: open.

## P4: Autoscaling when demand for one model exceeds capacity

Covers: Bounded Queue Wait Time, Capacity Scaling, Online Traffic Protection,
Wait-Time Visibility.

Status: open.
