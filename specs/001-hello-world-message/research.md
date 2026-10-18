# Phase 0 Research: Hello World Message Delivery

## Decision 1: Use AWS Step Functions Standard workflow as the only compute orchestration

- **Decision**: Implement the feature as a Step Functions state machine and avoid Lambda for greeting generation and routing logic.
- **Rationale**: This directly satisfies the requirement to prefer Step Functions over other compute forms, while keeping operational overhead low and preserving serverless-first architecture.
- **Alternatives considered**:
  - **AWS Lambda handler**: Rejected because the feature requirement explicitly prefers Step Functions.
  - **Step Functions Express**: Rejected for this scope because Standard provides clearer execution history and operational traceability for human-readable validation.

## Decision 2: Return greeting in execution output and enable CloudWatch execution logging

- **Decision**: Ensure the completed execution output contains the exact message `"Hello, World!"` and enable CloudWatch logging with execution data for observability.
- **Rationale**: The output is retrievable through `DescribeExecution` for completed Standard workflow runs; CloudWatch Logs provide an additional user-visible record. Logs are best-effort and event data can be truncated, so logs are not the only output path.
- **Alternatives considered**:
  - **Rely on CloudWatch Logs as the sole output channel**: Rejected because log delivery is best-effort and logged execution data can be truncated.
  - **Direct `cloudwatchlogs:PutLogEvents` SDK integration in the state machine**: Rejected due to unnecessary complexity (stream management and event timestamp handling) for this feature.
  - **External notifier service**: Rejected because CloudWatch visibility is already required and sufficient.

## Decision 3: Retry temporary delivery failures once before fallback

- **Decision**: Configure one retry for a temporary failure in a retry-capable Task, Parallel, or Map state; if the retry also fails, return a readable temporary-issue fallback message.
- **Rationale**: Step Functions supports explicit retry limits for these states. A retry limit of one means one retry after the initial attempt. The failure match must be limited to temporary delivery errors; fallback follows exhaustion.
- **Alternatives considered**:
  - **No retry or fallback output**: Rejected because it violates the clarified retry behavior and fallback acceptance scenario.
  - **Retry indefinitely**: Rejected because the requirement specifies one automatic retry and bounded failure handling.

## Clarifications Resolved

All technical context items are resolved for planning:

- Message format finalized as exact text: `"Hello, World!"`
- Compute approach finalized: Step Functions-first (no Lambda)
- User-observable channel finalized: Step Functions execution output or CloudWatch Logs
- Failure handling finalized: retry temporary delivery failures once automatically, then return fallback content
