---
name: plan-supervisor
description: "Maintain a persistent global plan for multi-step or parallelizable work, require boundary clarification when inputs are incomplete, dispatch each step to the right skill or subagent, and advance only after fresh verification. Use when a task needs a long-running plan, independent subtasks, step-by-step execution, or evidence-based recovery when progress stalls."
---

# Plan Supervisor

## Purpose

Maintain the global plan as the source of truth.

This skill does not implement steps itself. It decides the next step, delegates execution, checks verification, and updates plan state until the whole plan is done.
Prefer decomposition over monolithic execution.

## Relationship to Other Skills

1. `using-superpowers`
   - Skill routing and selection entry layer.
   - Decides which skill should be used first.
   - Use its routing rules before deciding whether to invoke a subagent and which skill it should use.

2. `dispatching-parallel-agents`
   - Parallel task fan-out and isolated subagent execution.
   - Use for independent tasks that can be worked on without shared state or sequential dependencies.

3. `writing-plans`
   - Plan decomposition and step granularity.
   - Use its task-breaking style, but keep this skill as the long-running controller.

4. `ai-change-closure`
   - Single-step implementation and verification loop.
   - Use for concrete step execution.

5. `evidence-first-debugging`
   - Failure diagnosis branch when a step loops without new evidence.

6. `verification-before-completion`
   - Completion gate.
   - Never advance or claim done without fresh verification evidence.

7. `delivery-orchestrator`
   - Final status and evidence sink after the plan is complete.

## Core Rule

A plan is complete only when every step has passed verification.

## When To Use

Use this skill when a task:

1. Needs a persistent global plan.
2. Must be executed step by step.
3. Can be decomposed into independent tasks that may run in parallel.
4. Needs automatic step selection or handoff between skills.
5. Should recover from repeated failure by switching to diagnosis.
6. Must end with verified completion, not just a written plan.
7. Does not come with an explicit plan and needs one created first.

Do not use this skill for one-off tasks that do not need plan state.

## Required Inputs

Collect or confirm:

1. Goal.
2. Non-goals.
3. Current context.
4. Constraints.
5. Acceptance criteria.
6. Available evidence.

If some inputs are missing, proceed only with the safest minimal assumption and mark the missing parts explicitly.
If goal, non-goals, acceptance criteria, or independent-task boundaries are unclear, stop and ask a clarifying question before drafting the plan.
If no explicit plan exists, create a minimal plan from the goal, constraints, and acceptance criteria before dispatching steps.
If the work contains multiple independent tasks, identify them before execution and prefer parallel decomposition.

## Plan State

The plan must track:

1. goal
2. steps
3. current_step
4. step_status
5. acceptance_criteria
6. required_checks
7. blockers
8. evidence
9. final_status

## Workflow

1. Intake.
   - Convert the request into explicit facts.
   - Separate confirmed facts from assumptions.

2. Plan creation.
   - If no explicit plan exists, draft the minimal plan first.
   - Keep it small, ordered, and verifiable.
   - Make the plan visible as the current source of truth before execution.

3. Plan construction.
   - Break the work into small, verifiable steps.
   - Keep each step minimal and ordered.
   - Define the test or check for each step before execution.
   - Identify independent tasks before execution; if steps are independent, split them into separate execution handoffs instead of merging unrelated work.
   - When file or module ownership is disjoint, prefer separate subagents or separate execution chunks.
   - Use `dispatching-parallel-agents` for independent tasks that can be worked on without shared state or sequential dependencies.

4. Process state.
   - Use git, Linear, scripts, or task notes for durable process tracking when available.
   - Do not force the LLM to remember process state that tools can track.

5. Testing.
   - Every independent task must have its own verification.
   - The merged result must also be verified after the subtasks are combined.
   - Cover three layers for code or content changes:
     - Single-item tests for the step's core behavior.
     - Integration tests for the step's interaction with surrounding pieces.
     - Real regression tests for the exact failure path or historical bug.
   - Prefer a regression test or regression script for bug fixes.
   - If a failure class has appeared before, require a reusable regression check or artifact instead of a one-off ad hoc check.
   - Prefer at least one positive case and one failure or edge case when adding or changing behavior.
   - If automated tests are not available, define the exact manual verification command or artifact.

6. Step dispatch.
   - Before assigning work to a subagent, follow `using-superpowers` for skill selection and subagent routing.
   - Use `dispatching-parallel-agents` to fan out independent tasks when they can proceed without shared state.
   - Send concrete implementation steps to `ai-change-closure`.
   - Send repeated failures without new evidence to `evidence-first-debugging`.
   - Send final results to `delivery-orchestrator`.

7. Verification gate.
   - Do not advance until the current step is verified with fresh evidence.
   - A step is not verified unless its required checks pass.

8. Plan advancement.
   - Update the plan state after each verified step.
   - Select the next step only after the current step is closed.

9. Execution handoff.
   - If a next step is ready and no blocker exists, return a concrete handoff for the execution skill in the same response.
   - Include the next step, required checks, and any missing inputs.
   - Do not wait for a separate activation if the current context already contains enough information to continue.
   - If continuous execution is not possible in the current environment, state the blocker explicitly.

10. Finalization.
   - Mark the plan done only after all steps pass verification.
   - Hand the evidence bundle to `delivery-orchestrator`.

## Step Statuses

Use:

1. `pending`
2. `in_progress`
3. `passed`
4. `blocked`
5. `failed`

## Decision Rules

1. Use `ai-change-closure` for a concrete code or content change.
2. Use `evidence-first-debugging` when the same step loops without new evidence.
3. Use `delivery-orchestrator` when the full plan is complete and needs final status capture.

## Stop Rules

Stop and switch out of the current path if:

1. No fresh verification is available.
2. Two rounds produce no new progress.
3. The current context is insufficient.
4. Dependencies or permissions are missing.
5. The user changes the goal.

## Output Contract

Always return:

1. Context summary.
2. Plan summary.
3. Current step.
4. Delegated skill.
5. Verification result.
6. Updated status.
7. Next step or blocker.
8. Execution handoff, when a next step is ready.
9. If the response includes a blocker, diagnosis, plan divergence, or merged-verification failure, include a concise Mermaid flowchart or decision tree that pinpoints where the problem is and which branch should happen next.
10. Do not force a diagram for routine status updates that do not involve diagnosis or branching.

## Prompt Template

Use this template when starting or updating a plan:

```text
Use plan-supervisor for this task.

Goal:
{{goal}}

Non-goals:
{{non_goals}}

Current context:
{{current_context}}

Constraints:
{{constraints}}

Acceptance criteria:
{{acceptance_criteria}}

Available evidence:
{{evidence}}
```
