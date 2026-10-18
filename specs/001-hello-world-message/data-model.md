# Data Model: Hello World Message

## Entity: GreetingRequest

Represents a user-initiated request to receive a greeting.

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| requestId | string | No | If present, min length 1 | Correlates execution and logs. |
| simulateFailure | boolean | No | Default `false` | Forces unavailable path for validation/testing. |

## Entity: GreetingResponse

Represents the user-facing result returned by workflow execution.

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| requestId | string | No | Mirrors input when provided | Maintains request correlation. |
| status | string | Yes | Enum: `SUCCESS`, `UNAVAILABLE` | Contracted outcome classification. |
| message | string | Yes | `SUCCESS` => exactly `Hello World!`; `UNAVAILABLE` => retry guidance | User-visible message body. |
| timestamp | string | Yes | ISO-8601 date-time | Execution completion marker. |

## Entity: GreetingExecutionLog

Represents the CloudWatch log record users read for greeting confirmation.

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| executionArn | string | Yes | Non-empty ARN | Step Functions execution identifier. |
| logLevel | string | Yes | Step Functions logging level (`ALL`) | Operational telemetry metadata. |
| output | object | Yes | Must contain `status` and `message` | Includes greeting payload when execution data logging is enabled. |

## Relationships

- One `GreetingRequest` produces one terminal `GreetingResponse`.
- One workflow execution emits multiple log events; at least one event contains
  terminal `GreetingResponse` output for CloudWatch inspection.

## State Transitions

1. `RECEIVED` -> `SUCCESS` when request is valid and normal path executes.
2. `RECEIVED` -> `UNAVAILABLE` when failure path is selected or delivery fails.
3. `UNAVAILABLE` -> `RECEIVED` on user retry via a new execution request.
