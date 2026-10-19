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
4. Resolve stack outputs for this feature:
   ```bash
   STACK_NAME=test-speckit-copilot-skills
   STATE_MACHINE_ARN=$(aws cloudformation describe-stacks \
     --stack-name "$STACK_NAME" \
     --query "Stacks[0].Outputs[?OutputKey=='HelloWorldMessageStateMachineArn'].OutputValue" \
     --output text)
   LOG_GROUP_NAME=$(aws cloudformation describe-stacks \
     --stack-name "$STACK_NAME" \
     --query "Stacks[0].Outputs[?OutputKey=='HelloWorldMessageLogGroupName'].OutputValue" \
     --output text)
   ```

## Validation Scenario 1: Manual greeting execution

1. Start execution manually:
   ```bash
   EXECUTION_ARN=$(aws stepfunctions start-execution \
     --state-machine-arn "$STATE_MACHINE_ARN" \
     --name hello-world-$(date +%s) \
     --input '{}' \
     --query executionArn \
     --output text)
   ```
2. Confirm execution status reaches `SUCCEEDED`.
   ```bash
   aws stepfunctions describe-execution \
     --execution-arn "$EXECUTION_ARN" \
     --query status \
     --output text
   ```

**Expected outcome**: Execution starts successfully with manual trigger and completes without requiring additional user input.

## Validation Scenario 2: Confirm CloudWatch message content

1. Read recent state machine logs:
   ```bash
   aws logs tail "$LOG_GROUP_NAME" --since 10m --format short
   ```
2. Verify at least one line contains the exact text:
   ```text
   Hello World!
   ```

**Expected outcome**: User can read the exact greeting text from CloudWatch logs.

## Validation Scenario 3: Retry and failure behavior

1. Temporarily block log writes on the state machine execution role so log-write calls fail:
   ```bash
   ROLE_NAME=$(aws cloudformation describe-stack-resources \
     --stack-name "$STACK_NAME" \
     --logical-resource-id HelloWorldMessageStateMachineRole \
     --query "StackResources[0].PhysicalResourceId" \
     --output text)
   aws iam put-role-policy \
     --role-name "$ROLE_NAME" \
     --policy-name HelloWorldMessageDenyLogsForValidation \
     --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Deny","Action":"logs:PutLogEvents","Resource":"*"}]}'
   ```
2. Start another execution and capture the new execution ARN:
   ```bash
   FAILED_EXECUTION_ARN=$(aws stepfunctions start-execution \
     --state-machine-arn "$STATE_MACHINE_ARN" \
     --name hello-world-failure-$(date +%s) \
     --input '{}' \
     --query executionArn \
     --output text)
   ```
3. Verify the execution reaches `FAILED` and inspect retry evidence in execution history:
   ```bash
   aws stepfunctions describe-execution \
     --execution-arn "$FAILED_EXECUTION_ARN" \
     --query status \
     --output text
   aws stepfunctions get-execution-history \
     --execution-arn "$FAILED_EXECUTION_ARN" \
     --max-results 200
   ```
4. Remove the temporary deny policy after validation:
   ```bash
   aws iam delete-role-policy \
     --role-name "$ROLE_NAME" \
     --policy-name HelloWorldMessageDenyLogsForValidation
   ```
5. Confirm CloudWatch evidence for failures (error messages) and the user-friendly terminal output in execution history.

**Expected outcome**:
- Up to 3 retries occur after the initial failure.
- If all 4 attempts fail, execution ends in `FAILED` with a user-friendly error message.

## Reference Artifacts

- Data model: `specs/001-hello-world-message/data-model.md`
- Interface contracts: `specs/001-hello-world-message/contracts/greeting-workflow-contract.md`
