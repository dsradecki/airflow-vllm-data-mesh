## MODIFIED Requirements

### Requirement: Job Submission

The system SHALL let a tenant submit a batch inference job through the
platform's submission interface with exactly two tenant-controlled inputs:
`model_name` and `data_path`. When a submission is valid, the system SHALL
accept it and return a job identifier that is unique across all tenants.

#### Scenario: Valid submission

- **WHEN** a tenant submits a job with a registered `model_name` and a `data_path`
- **THEN** the system accepts the job
- **AND** returns a unique job identifier
- **AND** the job is in state `QUEUED` or `RUNNING`
