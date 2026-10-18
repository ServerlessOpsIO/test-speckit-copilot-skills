# Quickstart: Validate Hello World Message Delivery

This guide validates the feature end-to-end once implementation is in place.

## Prerequisites

- AWS CLI configured with deploy and execution permissions
- AWS SAM CLI installed
- Access to CloudWatch Logs and Step Functions in target account/region

## 1. Validate and build

```bash
sam validate --lint
sam build
```

Expected outcome:

- Template passes validation
- Build succeeds with Step Functions resources packaged

## 2. Deploy stack

```bash
sam deploy --guided
```

Record the deployed state machine ARN and CloudWatch log group name from stack outputs/resources.

## 3. Run success scenario

Start execution with default input:

```bash
aws stepfunctions start-execution \
  --state-machine-arn <STATE_MACHINE_ARN> \
  --input '{}'
```

Describe execution after completion:

```bash
aws stepfunctions describe-execution \
  --execution-arn <EXECUTION_ARN>
```

Expected outcome:

- Execution status is `SUCCEEDED`
- Output contains `result.status = "SUCCESS"`
- Output contains exact `result.message = "Hello, World!"`

## 4. Confirm CloudWatch log message for success

Inspect log events for the execution:

```bash
aws logs filter-log-events \
  --log-group-name <STEP_FUNCTIONS_LOG_GROUP> \
  --filter-pattern <EXECUTION_ARN>
```

Expected outcome:

- Execution output provides the message directly; CloudWatch Logs provide additional visibility into execution data.
- If checking the logs, an execution log entry includes output data with `result.message` equal to `"Hello, World!"`.

## 5. Run fallback scenario

```bash
aws stepfunctions start-execution \
  --state-machine-arn <STATE_MACHINE_ARN> \
  --input '{"forceFallback": true}'
```

Expected outcome:

- Execution status is `SUCCEEDED`
- Output contains `result.status = "FALLBACK"`
- Output includes a retry-guidance fallback message
- Execution history shows one automatic retry before the fallback result.

## References

- Data model: [data-model.md](./data-model.md)
- Interface contract: [contracts/state-machine-contract.md](./contracts/state-machine-contract.md)
