<!--
Sync Impact Report
Version change: unratified scaffold -> 1.0.0 (initial constitution)
Modified principles: none; all five principles are newly established
Added sections: AWS Serverless Constraints; Development Workflow
Removed sections: none
Follow-up TODO: confirm the original ratification date
Remove this report before committing the amended constitution.
-->

# test-speckit-copilot-skills Constitution

## Core Principles

### I. Serverless-First Architecture
Workloads MUST use managed, event-driven AWS services and stateless, bounded Lambda
functions where those services meet the requirement. An always-on or self-managed
component MUST include a documented rationale explaining why a managed serverless
alternative is unsuitable. This keeps operational burden and idle cost proportionate
to the workload.

### II. Least Privilege and Secure Boundaries
Every workload identity MUST have only the actions and resource access it needs.
Wildcard permissions MUST be avoided unless technically necessary and explicitly
justified. Inputs crossing trust boundaries MUST be validated; sensitive values MUST
not be committed to source or exposed in logs; data MUST use encryption in transit
and at rest where supported. These controls reduce the impact of compromised
identities and malformed events.

### III. Infrastructure as Code
AWS resources and their configuration MUST be declared and reviewed in versioned
infrastructure-as-code templates, using AWS SAM and CloudFormation conventions
already present in the project. Changes MUST NOT rely on undocumented console edits;
required environment-specific values MUST be parameterized, and secrets MUST be
stored in an appropriate managed secret store rather than template defaults.
Declarative infrastructure makes deployments repeatable and reviewable.

### IV. Tested Contracts and Changes
Behavioral changes MUST include or update automated tests for the affected behavior.
Event handlers and service integrations MUST be tested against their documented
input and output contracts. Changes to AWS resource configuration MUST include
validation appropriate to the modified resources. Tests and validation MUST pass
before merge, except for an explicitly approved and recorded waiver.

### V. Operational Readiness
Production workloads MUST provide structured, actionable logs and observable
failure signals. Event-driven processing MUST define appropriate retry,
idempotency, and dead-letter or failure-destination behavior for its trigger and
failure modes. Resource limits, timeouts, and concurrency MUST be selected for the
workload rather than left accidental. These requirements make failures diagnosable
and recovery predictable.

## AWS Serverless Constraints

AWS Lambda functions MUST declare an explicit runtime, timeout, and memory
configuration appropriate to their workload. Permissions, event sources, and
service resources MUST be defined in the infrastructure template. Deployments to
production MUST use a reviewed infrastructure change and an approved deployment
process; changes MUST NOT bypass repository review or deployment safeguards.

## Development Workflow

For each change, reviewers MUST verify its consistency with these principles,
including the relevant tests, infrastructure definitions, permissions, and
operational behavior. Before merge, the project MUST run the applicable automated
tests and infrastructure validation available in its toolchain. Production
deployment changes MUST be reviewed before execution, and any exception to this
workflow MUST be documented with its scope and approval.

## Governance

This constitution governs project architecture and delivery requirements.
Amendments MUST be made through a reviewed repository change that explains the
reason for the amendment and its impact on existing work. Reviewers MUST assess
whether dependent specifications, plans, and implementation guidance need updates.

The constitution uses semantic versioning:
- MAJOR increments for backward-incompatible governance changes, including removal
  or redefinition of a principle.
- MINOR increments when a principle or governance requirement is added or materially
  expanded.
- PATCH increments for clarifications and non-semantic wording changes.

Compliance MUST be reviewed during planning and code review, and before production
deployment when the change affects infrastructure or runtime behavior. A deviation
MUST identify the unmet requirement, rationale, risk, approver, and any corrective
action or expiry. Waivers do not amend this constitution.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date is not recorded | **Last Amended**: 2026-10-16
