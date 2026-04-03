# Plan Supervisor

This repository hosts the `plan-supervisor` skill that keeps a long-running plan on track, ensures scopes are decomposed, and enforces fresh verification before advancing. It leans on `using-superpowers` for routing decisions, `dispatching-parallel-agents` for parallel work, and `ai-change-closure` for concrete steps.

## Key Behaviors

- Require boundary confirmation when inputs (goal, non-goals, tasks) are unclear.
- Split independent steps into separate execution handoffs and fan them out with `dispatching-parallel-agents`.
- Each step and the merged result must pass single-item, integration, and regression checks before moving on.
- Blockers, diagnostics, or merged-verification failures mandate a Mermaid flowchart/decision tree that pinpoints the stuck branch.

## Structure

- `SKILL.md`: human-readable skill documentation loaded by Codex/Codex CLI.
- `agents/openai.yaml`: metadata for the agent (display name, description, default prompt).

## Validation

Use the skill-creator quick validate script to ensure the documentation stays valid (adjust the path to the Python interpreter that has the required dependencies):

```bash
python ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ~/.agents/skills/plan-supervisor
```

## Workflow Suggestions

1. Start with `plan-supervisor` for planning.
2. Confirm boundaries via `ask-user-question`.
3. Fan out parallel tasks through `dispatching-parallel-agents`.
4. Send concrete steps to `ai-change-closure`.
5. Visualize blockers/diagnostics using Mermaid flowcharts in the final response.
