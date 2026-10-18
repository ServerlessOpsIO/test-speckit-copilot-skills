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

- `forceFallback` is optional and defaults to `false`.
- `requestId` is optional and used only for correlation/traceability.
- Additional unknown fields should be ignored by workflow logic unless explicitly mapped in implementation.

## Output Contract

### Success Output

```json
{
  "result": {
    "status": "SUCCESS",
    "message": "Hello World!",
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
- For successful runs, CloudWatch log events must contain output payload data where `result.message` is exactly `"Hello World!"`.
- For fallback runs, CloudWatch log events must contain `status` as `FALLBACK` and a retry-guidance message.

## Error and Fallback Semantics

- Workflow must not return empty output payloads.
- If success path cannot produce the required greeting message, workflow emits fallback payload instead of blank response.
