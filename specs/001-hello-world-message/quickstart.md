# Quickstart: Validate Hello World Message Delivery

This guide validates the feature end-to-end for manual Step Functions execution and CloudWatch log visibility.

## Prerequisites

1. AWS CLI configured with credentials that can deploy SAM, start Step Functions executions, and read CloudWatch logs.
2. SAM CLI installed.
3. Target AWS account/region selected (`AWS_PROFILE` / `AWS_REGION`).

## Setup

1. Validate template syntax and linting:
   ```bash
   sam validate --lint
   ```
2. Build the application:
   ```bash
   sam build
   ```
3. Deploy with required parameters:
   ```bash
   sam deploy --guided
   ```

## Validation Scenario 1: Manual greeting execution

1. Start execution manually:
   ```bash
   aws stepfunctions start-execution \
     --state-machine-arn <STATE_MACHINE_ARN> \
     --name hello-world-$(date +%s) \
     --input '{}'
   ```
2. Confirm execution status reaches `SUCCEEDED`.

**Expected outcome**: Execution starts successfully with manual trigger and completes without requiring additional user input.

## Validation Scenario 2: Confirm CloudWatch message content

1. Read recent state machine logs:
   ```bash
   aws logs tail <LOG_GROUP_NAME> --since 10m --format short
   ```
2. Verify at least one line contains the exact text:
   ```text
   Hello World!
   ```

**Expected outcome**: User can read the exact greeting text from CloudWatch logs.

## Validation Scenario 3: Retry and failure behavior

1. Simulate transient logging failure conditions (for implementation test path) and re-run execution.
2. Observe retry attempts in execution history/logs.
3. Verify terminal behavior after the fourth failed attempt.

**Expected outcome**:
- Up to 3 retries occur after the initial failure.
- If all 4 attempts fail, execution ends in `FAILED` with a user-friendly error message.

## Reference Artifacts

- Data model: `specs/001-hello-world-message/data-model.md`
- Interface contracts: `specs/001-hello-world-message/contracts/greeting-workflow-contract.md`
