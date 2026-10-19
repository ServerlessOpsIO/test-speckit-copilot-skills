# Contract: Greeting Workflow External Interface

## Interface 1: Manual Step Functions Trigger

### Operation

- **API**: `StartExecution`
- **Trigger mode**: Manual only (Console, CLI, SDK)

### Request Contract

```json
{
  "stateMachineArn": "arn:aws:states:<region>:<account-id>:stateMachine:HelloWorldMessageStateMachine",
  "name": "hello-world-<unique-suffix>",
  "input": "{}"
}
```

### Request Rules

- `stateMachineArn` is required and must reference the deployed workflow.
- `name` is required and must be unique per active execution window.
- `input` must be valid JSON object text. Empty object (`{}`) is supported and expected.

### Response Contract (Immediate Start Acknowledgement)

```json
{
  "executionArn": "arn:aws:states:<region>:<account-id>:execution:HelloWorldMessageStateMachine:hello-world-1234567890",
  "startDate": "2026-10-18T21:00:00.000Z"
}
```

### Error Contract (Trigger Failure)

When trigger is unavailable or malformed, the user receives a clear error from the invocation interface (for example invalid ARN, invalid input, or access denied), and no execution is started.

## Interface 2: CloudWatch Log Outcome

### Success Log Contract

At least one CloudWatch log event for a successful execution contains:

```text
Hello World!
```

### Failure Contract

If all 4 attempts fail, execution ends in `FAILED` and logs contain a user-friendly failure message indicating the greeting could not be processed.

## Retry Contract

- Initial attempt + up to 3 retries (maximum 4 total attempts).
- Retries apply to transient logging failures only.
- Terminal failure occurs immediately after the fourth failed attempt.
