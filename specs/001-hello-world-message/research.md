# Research: Hello World Message Delivery

## Decision 1: Use AWS Step Functions as the only compute orchestrator

- **Decision**: Implement the greeting flow as an AWS Step Functions state machine and avoid Lambda for this feature.
- **Rationale**: The requirement explicitly prefers Step Functions over other compute options, and the workflow is small enough to model directly in state transitions and service integrations.
- **Alternatives considered**:
  - **Lambda-only handler**: Rejected because it conflicts with the explicit Step Functions preference.
  - **API Gateway + Lambda**: Rejected as unnecessary for manually triggered executions.

## Decision 2: Trigger executions manually via `StartExecution`

- **Decision**: Use manual invocation through Step Functions `StartExecution` (AWS Console/CLI/SDK), with no automatic event source.
- **Rationale**: The feature requires manual triggering; `StartExecution` is the canonical direct trigger and keeps the system simple.
- **Alternatives considered**:
  - **EventBridge schedule**: Rejected because it is automatic, not manual.
  - **SQS/EventBridge event trigger**: Rejected because this introduces an unnecessary producer dependency.

## Decision 3: Write greeting directly to CloudWatch Logs

- **Decision**: Emit a CloudWatch log entry with the exact text `Hello World!` from the workflow path.
- **Rationale**: Users must read the message in CloudWatch logs; direct log emission satisfies that requirement without adding a data store or downstream service.
- **Alternatives considered**:
  - **Return-only execution output**: Rejected because the requirement is explicit about CloudWatch logs.
  - **Persist to DynamoDB/S3 then mirror to logs**: Rejected as unnecessary scope and complexity.

## Decision 4: Model retries and terminal failure in workflow definition

- **Decision**: Configure retry policy for transient log-write failures with 3 retries after the initial attempt (4 total), followed by a fail state that returns a clear error.
- **Rationale**: Matches FR-005 and preserves deterministic, inspectable behavior in the state machine.
- **Alternatives considered**:
  - **No retries**: Rejected because it violates FR-005.
  - **Unbounded retries**: Rejected because it risks runaway executions and unclear user outcomes.

## Decision 5: Use Step Functions Standard workflow type

- **Decision**: Use a Standard state machine.
- **Rationale**: Standard workflows provide durable execution history and straightforward operational troubleshooting for this feature's reliability and observability goals.
- **Alternatives considered**:
  - **Express workflow**: Rejected because long-term execution visibility and debugging clarity are preferred over high-throughput optimization for this low-volume manual flow.
