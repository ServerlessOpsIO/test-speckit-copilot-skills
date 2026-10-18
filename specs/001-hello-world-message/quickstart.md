# Quickstart: Validate Hello World Step Functions Flow

This guide validates end-to-end behavior for the Step Functions-first greeting
design, including CloudWatch log visibility of the output message.

## Prerequisites

- AWS CLI configured for the target account and region
- AWS SAM CLI installed
- Deployment permissions for CloudFormation, Step Functions, IAM, and CloudWatch Logs
- A deployed stack containing:
  - Step Functions state machine resource for this feature
  - CloudWatch log group configured as the state machine log destination

## Reference Contracts

- Input contract: [`contracts/start-execution-input.schema.json`](./contracts/start-execution-input.schema.json)
- Output contract: [`contracts/state-machine-output.schema.json`](./contracts/state-machine-output.schema.json)
- Data model: [`data-model.md`](./data-model.md)

## 1. Validate and deploy infrastructure

```bash
sam validate --lint
sam build
sam deploy --guided
```

Expected outcome:
- Template validates and deploys successfully.
- Deployed state machine has logging enabled with execution data included.

## 2. Run success-path execution

```bash
STATE_MACHINE_ARN="<deployed-state-machine-arn>"
aws stepfunctions start-execution \
  --state-machine-arn "$STATE_MACHINE_ARN" \
  --name "hello-world-success-$(date +%s)" \
  --input '{"requestId":"req-success-001"}'
```

Expected outcome:
- Execution reaches `SUCCEEDED`.
- Final output contains:
  - `"status":"SUCCESS"`
  - `"message":"Hello World!"`

## 3. Run unavailable-path execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn "$STATE_MACHINE_ARN" \
  --name "hello-world-unavailable-$(date +%s)" \
  --input '{"requestId":"req-fail-001","simulateFailure":true}'
```

Expected outcome:
- Execution reaches `SUCCEEDED` with unavailable response payload.
- Final output contains:
  - `"status":"UNAVAILABLE"`
  - `"message":"Greeting is currently unavailable. Please retry."`

## 4. Verify CloudWatch logs include greeting output

```bash
LOG_GROUP_NAME="<step-functions-log-group>"
aws logs tail "$LOG_GROUP_NAME" --since 5m --format short
```

Expected outcome:
- Log entries for recent executions are present.
- Execution data includes `message` field values from both success and unavailable
  responses.
- Users with CloudWatch Logs read access can view `Hello World!` in log output.
