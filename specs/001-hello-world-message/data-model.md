# Data Model: Hello World Message Delivery

## Entity: GreetingExecutionRequest

- **Description**: Represents a user's manual request to run the greeting workflow.
- **Fields**:
  - `executionName` (string, required): Unique execution identifier used with `StartExecution`.
  - `input` (object, optional): Must default to `{}`; no business fields are required.
  - `triggerType` (string, derived): Always `manual`.
- **Validation rules**:
  - Request is invalid if state machine ARN is unavailable or malformed.
  - Request input must be valid JSON object; empty object is valid.

## Entity: GreetingWorkflowContext

- **Description**: Runtime context carried through state transitions.
- **Fields**:
  - `message` (string): Constant value `Hello World!`.
  - `attempt` (number): Starts at 1 and increments on retries.
  - `maxAttempts` (number): Constant value `4` (initial + 3 retries).
  - `error` (object, optional): Captured transient/final error details.
- **Validation rules**:
  - `message` must exactly equal `Hello World!`.
  - `attempt` must be between `1` and `4`, inclusive.

## Entity: GreetingLogRecord

- **Description**: CloudWatch log event produced by a successful greeting execution.
- **Fields**:
  - `timestamp` (ISO-8601 string): Event creation time.
  - `executionArn` (string): Step Functions execution identifier.
  - `level` (string): `INFO` for successful log event.
  - `message` (string): Exact text `Hello World!`.
- **Validation rules**:
  - `message` is case-sensitive and must include the exclamation mark.

## Entity: GreetingExecutionResult

- **Description**: Final user-visible outcome for a workflow execution.
- **Fields**:
  - `status` (enum): `SUCCEEDED` or `FAILED`.
  - `message` (string): Success message (`Hello World!`) or user-friendly error.
  - `attempts` (number): Number of attempts made (1..4).
  - `failureReason` (string, optional): Included only when status is `FAILED`.
- **Validation rules**:
  - On success, `status` must be `SUCCEEDED` and `message` must be `Hello World!`.
  - On terminal failure, `status` must be `FAILED` with a clear user-friendly error message.

## State Transitions

1. `REQUESTED` -> `RUNNING` when manual `StartExecution` is accepted.
2. `RUNNING` -> `LOGGING` when workflow begins message emission.
3. `LOGGING` -> `SUCCEEDED` when log write succeeds.
4. `LOGGING` -> `RETRYING` on transient log failure and attempts remaining.
5. `RETRYING` -> `LOGGING` until `attempt` reaches `4`.
6. `LOGGING` -> `FAILED` when the final attempt fails.
