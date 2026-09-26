---
name: implement-plan
description: Implement technical plans from thoughts/shared/plans with verification
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The text the user typed after `/implement_plan` in the triggering message **is** the plan path. Assume you always have it available in this conversation even if `$ARGUMENTS` appears literally below. Do not ask the user to repeat it unless they provided an empty command.

Given that plan path, do this:

# Implement Plan (GitHub Copilot Instructions)

Purpose: Provide clear, deterministic instructions for GitHub Copilot to implement approved technical plans from `thoughts/shared/plans/` with phase-by-phase verification and human checkpoints.

## Core Principles
1. Always read the plan file and all referenced files COMPLETELY before starting—never truncate or skim.
2. Follow the plan's intent while adapting to actual codebase state.
3. Implement each phase fully before moving to the next.
4. Update checkboxes in the plan as sections complete.
5. Verify work against success criteria before proceeding.
6. Pause for human verification after each phase (unless batch execution requested).
7. Never fabricate file paths, function names, or claim completion without evidence.
8. Work on the current branch provided by the developer—do NOT create new branches.

## Implementation Workflow
1. **Load Plan & Context**
   - Read the plan file completely from `thoughts/shared/plans/<plan-path>`.
   - Identify existing checkmarks (`- [x]`) to determine progress.
   - Read the original ticket/issue referenced in the plan.
   - Read ALL files mentioned in the plan fully (no partial reads).
   - Create a todo list reflecting the plan's phases and tasks.

2. **Understand Before Acting**
   - Map relationships between files and components involved.
   - Identify dependencies between phases.
   - Note any constraints, edge cases, or validation rules in the plan.
   - If unclear, ask for clarification before proceeding.

3. **Execute Phase-by-Phase**
   - Implement all changes specified for the current phase.
   - Follow existing code patterns and conventions in the codebase.
   - Update imports, exports, and type definitions as needed.
   - Mark completed items in the plan file with `- [x]`.

4. **Verify Each Phase**
   - Run success criteria checks (see commands below).
   - Fix any lint, type, or test failures before proceeding.
   - Update todo list to reflect completion.
   - Check off items in the plan file using edit tools.

5. **Human Checkpoint**
   - After automated verification passes, pause and request manual verification.
   - Proceed to next phase only after human confirms.

## Handling Plan Mismatches
When reality diverges from the plan, STOP and present clearly:

```
Issue in Phase [N]:
Expected: <what the plan specifies>
Found: <actual codebase state>
Impact: <why this matters for implementation>

Options:
1. <proposed adaptation>
2. <alternative approach>

How should I proceed?
```

Do NOT silently deviate. Do NOT guess. Ask.

## Phase Completion Format
After completing automated verification for a phase:

```
Phase [N] Complete - Ready for Manual Verification

Automated verification passed:
- <check 1> ✓
- <check 2> ✓

Manual verification required (from plan):
- <manual step 1>
- <manual step 2>

Awaiting confirmation to proceed to Phase [N+1].
```

If instructed to execute multiple phases consecutively, skip intermediate pauses—only pause after the final requested phase.

## Resuming Interrupted Work
When the plan has existing checkmarks:
- Trust completed work as done.
- Resume from first unchecked item.
- Re-verify previous work only if something seems inconsistent.
- State clearly where you are resuming from.

## Success Criteria Verification
Verification commands for this repository:

- **Backend**: `cd backend && uv run ruff format` → `uv run pylint` → `uv run pyright` → `uv run pytest`
- **Frontend**: `cd frontend && pnpm exec prettier` → `pnpm exec eslint` → `pnpm exec jest`
- **E2E**: `nx agent-e2e`
- **Full stack**: Run both backend and frontend checks

Always run the checks specified in the plan's success criteria section.

## Todo List Management
Maintain a todo list that mirrors the plan structure:
- One todo per actionable item in the plan.
- Mark in-progress when starting a task.
- Mark completed immediately upon finishing.
- Keep todos synchronized with plan checkboxes.

## Citation Rules
When referencing implementation details:
- Cite file paths with line ranges: `path/to/file.py:42-57`
- Use backticks for all paths and code references.
- Reference specific functions/classes by name with their location.

## Prohibited
- Skipping phases or reordering without explicit approval.
- Marking manual verification steps complete without human confirmation.
- Fabricating test results or claiming checks passed without running them.
- Making silent deviations from the plan.
- Partial file reads when full context is required.
- Announcing tool names to the user.
- Creating new Git branches or switching branches—work on the current branch only.

## When No Plan Path Provided
If `$ARGUMENTS` is empty or no plan path is given:
```
No plan path provided.

Please specify the plan to implement:
- Provide the path relative to `thoughts/shared/plans/`
- Example: `/implement_plan feature-x/implementation-plan.md`

Available plans can be found in `thoughts/shared/plans/`.
```

## When Stuck
If implementation is blocked:
1. Re-read all relevant code for missed context.
2. Check if codebase evolved since plan was written.
3. Present the blocker clearly with what you've tried.
4. Suggest concrete next steps or questions.

Use sub-agents sparingly—only for targeted debugging or exploring unfamiliar subsystems.

## Style Guide
- Be concise; remove hedging language.
- Distinguish completed work from in-progress.
- Use active voice.
- Report progress factually with evidence.

---
Use this file as the single source of truth for implementing technical plans within this repository context.
