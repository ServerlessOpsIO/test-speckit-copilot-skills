# Contract: Hello World Step Functions Execution

## Interface

- **Interface Type**: AWS Step Functions `StartExecution`
- **State Machine Behavior**: Returns deterministic greeting payload and logs execution data to CloudWatch

## Input Contract

### JSON Shape

```json
{
  "forceFallback": false,
  "requestId": "optional-correlation-id"
}
```

### Rules

- `forceFallback` is optional and defaults to `false`; when `true`, it simulates a temporary delivery failure on both the initial attempt and the single retry so the fallback can be validated.
- `requestId` is optional and used only for correlation/traceability.
- Additional unknown fields should be ignored by workflow logic unless explicitly mapped in implementation.

## Output Contract

### Success Output

```json
{
  "result": {
    "status": "SUCCESS",
    "message": "Hello, World!",
    "retryable": false
  }
}
```

### Fallback Output

```json
{
  "result": {
    "status": "FALLBACK",
    "message": "Greeting temporarily unavailable. Please retry.",
    "retryable": true
  }
}
```

## CloudWatch Logging Contract

- Step Functions logging is enabled with execution data included.
- For successful runs, the execution output must contain `result.message` exactly `"Hello, World!"`; CloudWatch logs provide additional visibility into execution data.
- For fallback runs, CloudWatch log events must contain `status` as `FALLBACK` and a retry-guidance message.
- CloudWatch logging is best-effort; completed Standard workflow execution output is also retrievable with `DescribeExecution`.

## Error and Fallback Semantics

- Workflow must not return empty output payloads.
- A temporary delivery failure is retried once automatically.
- If delivery still fails after that retry, workflow emits the fallback payload instead of blank output.
