# Tasks: Hello World Message Delivery

**Input**: Design documents from `/specs/001-hello-world-message/`

**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, contracts/, quickstart.md

**Tests**: No automated Step Functions behavior tests are generated under the approved feature-scoped Principle IV waiver in `plan.md`. Required validation remains `sam validate --lint` and the manual quickstart execution scenarios, including success, CloudWatch message, and retry/failure checks.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story label (`[US1]`, `[US2]`, etc.) for story-phase tasks only
- Every task includes an exact file path

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare project files and deployment metadata for implementation.

- [ ] T001 Create the Step Functions definition scaffold in `statemachines/hello-world.asl.json` with top-level `StartAt`, `States`, and placeholder success/failure states.
- [ ] T002 [P] Add feature-specific deployment parameter values in `cfn-parameters.json` for the stack used to deploy this workflow.
- [ ] T003 [P] Add/confirm deployment tags for this feature in `cfn-tags.json` so stack resources are traceable during rollout.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define baseline infrastructure that all story work depends on.

**⚠️ CRITICAL**: Complete this phase before starting user story implementation.

- [ ] T004 Add `HelloWorldMessageLogGroup` resource in `template.yaml` with explicit retention and encryption settings for workflow observability.
- [ ] T005 Add `HelloWorldMessageStateMachine` resource in `template.yaml` as `AWS::Serverless::StateMachine` with `StateMachineType: STANDARD` and `DefinitionUri: statemachines/hello-world.asl.json`.
- [ ] T006 Configure least-privilege logging permissions and state machine logging configuration in `template.yaml` so the workflow can write required log events.
- [ ] T007 Add outputs in `template.yaml` for state machine ARN and log group name used by manual invocation and quickstart validation.

**Checkpoint**: Infrastructure foundation is ready; user story work can begin.

---

## Phase 3: User Story 1 - Receive a greeting (Priority: P1) 🎯 MVP

**Goal**: Users manually trigger the workflow and can read the exact greeting message in CloudWatch logs, including defined retry/failure behavior.

**Independent Test**: Deploy the stack, run `StartExecution`, confirm execution succeeds, and verify CloudWatch logs contain exact `Hello World!`; for failure simulation, verify retries cap at 4 total attempts and terminal failure is user-friendly.

### Implementation for User Story 1

- [ ] T008 [US1] Implement request/context initialization states in `statemachines/hello-world.asl.json` so `input` constraint is enforced verbatim: "`input` (object, optional): Must default to `{}`; no business fields are required.", and "`triggerType` (string, derived): Always `manual`."
- [ ] T009 [US1] Implement context constants in `statemachines/hello-world.asl.json` with verbatim constraints: "`message` must exactly equal `Hello World!`.", "`attempt` must be between `1` and `4`, inclusive.", and "`maxAttempts` (number): Constant value `4` (initial + 3 retries)."
- [ ] T010 [US1] Implement the CloudWatch log-write task path in `statemachines/hello-world.asl.json` to satisfy GreetingLogRecord constraint verbatim: "`message` is case-sensitive and must include the exclamation mark."
- [ ] T011 [US1] Implement retry loop and attempt progression in `statemachines/hello-world.asl.json` with verbatim retry contract: "Initial attempt + up to 3 retries (maximum 4 total attempts)." and "Retries apply to transient logging failures only."
- [ ] T012 [US1] Implement terminal success and failure states in `statemachines/hello-world.asl.json` with verbatim result constraints: "On success, `status` must be `SUCCEEDED` and `message` must be `Hello World!`." and "On terminal failure, `status` must be `FAILED` with a clear user-friendly error message."
- [ ] T013 [P] [US1] Update the external interface details in `specs/001-hello-world-message/contracts/greeting-workflow-contract.md` so request/response examples align with deployed template outputs and logical names.
- [ ] T014 [P] [US1] Update execution commands in `specs/001-hello-world-message/quickstart.md` to reference the actual output names from `template.yaml` for manual trigger and CloudWatch log checks.

**Checkpoint**: User Story 1 is independently functional and testable.

---

## Phase 4: Polish & Cross-Cutting Concerns

**Purpose**: Align cross-document consistency and finalize operational guidance.

- [ ] T015 [P] Normalize greeting text consistency across `specs/001-hello-world-message/spec.md` and `specs/001-hello-world-message/contracts/greeting-workflow-contract.md` to one canonical exact string.
- [ ] T016 Update `specs/001-hello-world-message/quickstart.md` with explicit retry/failure verification steps tied to execution history and CloudWatch evidence.
- [ ] T017 [P] Add feature usage notes in `README.md` for deploy, manual start, and log validation flow.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies.
- **Phase 2 (Foundational)**: Depends on Phase 1 completion.
- **Phase 3 (US1)**: Depends on Phase 2 completion.
- **Phase 4 (Polish)**: Depends on Phase 3 completion.

### User Story Dependencies

- **US1 (P1)**: Starts after Foundational phase; no dependency on other user stories.

### Within User Story 1

- Sequence for state machine core: `T008 -> T009 -> T010 -> T011 -> T012`
- Documentation alignment tasks `T013` and `T014` can run in parallel after `T012`.

---

## Parallel Opportunities

- **Phase 1**: `T002` and `T003` can run concurrently.
- **US1 docs**: `T013` and `T014` can run concurrently once ASL behavior is finalized.
- **Polish**: `T015` and `T017` can run concurrently.

---

## Parallel Example: User Story 1

```bash
# After T012 completes, execute documentation tasks in parallel:
Task: "T013 Update specs/001-hello-world-message/contracts/greeting-workflow-contract.md"
Task: "T014 Update specs/001-hello-world-message/quickstart.md"
```

---

## Implementation Strategy

### MVP First (US1 Only)

1. Complete Phase 1 (Setup).
2. Complete Phase 2 (Foundational).
3. Complete Phase 3 (US1).
4. Validate US1 independently using quickstart scenarios.

### Incremental Delivery

1. Deploy baseline infrastructure (Phases 1-2).
2. Deliver US1 end-to-end (Phase 3) and validate.
3. Apply consistency and operational polish (Phase 4).

### Team Parallelization

1. One developer finalizes `template.yaml` foundational resources.
2. One developer finalizes `statemachines/hello-world.asl.json`.
3. One developer handles docs (`contracts/`, `quickstart.md`, `README.md`) after behavior is stable.

---

## Notes

- All tasks use strict checklist format with IDs, optional `[P]`, and `[US1]` labels only for story-phase tasks.
- User story mapping prioritizes independent testability and MVP-first delivery.
