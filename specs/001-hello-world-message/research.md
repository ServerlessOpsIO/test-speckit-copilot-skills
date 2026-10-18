# Research: Hello World Message

## Decision 1: Use AWS Step Functions as the workflow compute layer

- **Decision**: Implement the greeting behavior in an AWS Step Functions state
  machine rather than AWS Lambda.
- **Rationale**: The feature explicitly prefers Step Functions over Lambda. The
  use case is simple orchestration with deterministic output, which Step Functions
  handles natively without additional runtime management.
- **Alternatives considered**:
  - AWS Lambda handler returning the greeting payload.
  - API Gateway direct mock response without state orchestration.

## Decision 2: Emit `Hello World!` through state output and CloudWatch execution logs

- **Decision**: Generate the greeting payload in workflow states and rely on Step
  Functions logging configuration (`IncludeExecutionData: true`) to write the
  greeting into CloudWatch Logs.
- **Rationale**: This keeps compute serverless and avoids custom log-write code.
  Users can read execution logs directly in CloudWatch, and the greeting payload
  remains traceable by execution.
- **Alternatives considered**:
  - Step Functions AWS SDK integration calling CloudWatch Logs APIs directly.
  - Lambda-based explicit log writing.

## Decision 3: Define a retryable unavailable outcome in the contract

- **Decision**: Model success and unavailable outcomes with explicit `status` and
  `message` fields; unavailable responses instruct users to retry.
- **Rationale**: This aligns with failure requirements while keeping output shape
  stable and testable.
- **Alternatives considered**:
  - Returning raw Step Functions errors without a user-facing message.
  - Omitting failure contract details from Phase 1 design.

## Decision 4: Keep interface contract at Step Functions StartExecution boundary

- **Decision**: Treat `StartExecution` input/output payload shape as the external
  interface contract for this feature.
- **Rationale**: The repository is currently infrastructure-centric and does not
  yet expose an API layer. StartExecution is the simplest user entry point that
  still satisfies request/response and logging validation.
- **Alternatives considered**:
  - Introducing API Gateway contract in this phase.
  - Using EventBridge-triggered asynchronous flow only.
