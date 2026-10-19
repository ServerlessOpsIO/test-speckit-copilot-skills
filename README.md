# Backstage / test-speckit-copilot-skills

test

## New Project Getting Started
This repository was generated from a template intended to get a new RDS database up running quickly. It uses the [AWS Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/) to define the infrastructure as code.

## Hello World workflow usage

1. Validate and deploy:
   ```bash
   sam validate --lint
   sam build
   sam deploy --guided
   ```
2. Resolve stack outputs and start an execution:
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
   aws stepfunctions start-execution \
     --state-machine-arn "$STATE_MACHINE_ARN" \
     --name hello-world-$(date +%s) \
     --input '{}'
   ```
3. Validate logs contain the exact greeting:
   ```bash
   aws logs tail "$LOG_GROUP_NAME" --since 10m --format short
   ```

Detailed manual validation steps (including retry/failure verification) are documented in `specs/001-hello-world-message/quickstart.md`.
