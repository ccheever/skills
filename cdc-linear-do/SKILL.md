---
name: cdc-linear-do
description: Execute a Linear issue end to end from an issue key such as ENG-22195. Use when the user invokes $cdc-linear-do, asks to do or complete a Linear task by ID, or wants Codex to work from main in a Git worktree, keep Linear updated, file follow-up Linear issues, verify the fix or task, self-review, commit, merge into main, and push origin main.
---

# CDC Linear Do

## Overview

Use this skill to turn a short Linear issue key into a complete coding workflow. Treat a request like `Use $cdc-linear-do ENG-22195` as:

```text
Please do ENG-22195 for me. Work in a worktree off of main. Keep Linear up to date with your progress and anything else that I should be updated on. File other Linear issues if you encounter them along the way. Verify your work has done what you were trying to do. When you've verified that, review your own code and fix any issues you find. Then do a commit. You can make commits in the worktree along the way if it makes sense. Then merge the commit into main and push to origin main.
```

## Workflow

### 1. Parse And Load The Linear Issue

- Extract the issue key from the user request. Accept keys like `ENG-22195`; if multiple keys are present, choose the explicitly requested task or ask a concise clarification.
- Use the available Linear integration to fetch the issue title, description, comments, status, assignee, project, labels, and linked issues. If Linear tools are deferred, use the active tool-discovery mechanism such as `tool_search` before continuing.
- Post an initial Linear update saying work has started, including the repository, planned worktree branch, and any immediate ambiguity or risk.
- If the issue does not contain enough information to identify the repository or expected behavior, inspect local context first. Ask the user only when the missing information cannot be inferred safely.

### 2. Create A Worktree From Main

- Find the repository root with `git rev-parse --show-toplevel`.
- Inspect `git status --short --branch` before changing anything. Do not overwrite or revert unrelated local changes.
- Fetch the latest main branch with `git fetch origin main` when a remote exists.
- Create a sibling worktree from latest main, using a branch name based on the Linear key, for example:

```bash
git worktree add -b eng-22195 ../<repo>-eng-22195 origin/main
```

- If the repository has no `origin/main`, fall back to local `main` and note that in Linear.
- Work only inside the task worktree until the merge step.

### 3. Implement The Issue

- Build enough context from the issue, code, tests, logs, and local docs to define the expected outcome.
- Keep the task scoped to the Linear issue. If unrelated defects, missing cleanup, or follow-on work appears, file separate Linear issues with reproduction steps, impact, and links back to the active issue.
- Update Linear when meaningful progress occurs: after context is understood, when implementation starts, when blocked, when verification begins, and before final merge.
- Make intermediate commits in the worktree when they help preserve clean checkpoints.

### 4. Verify The Work

- Run the smallest reliable test or command that proves the requested bug fix, feature, migration, or task works.
- Add or update tests when the change has stable behavior worth protecting.
- Run broader checks when the touched area is shared, user-facing, security-sensitive, or likely to regress.
- If a command cannot be run, explain why in Linear and in the final response. Do not present unrun checks as passing.

### 5. Self-Review And Fix

- Review the final diff against main, for example `git diff origin/main...HEAD` when `origin/main` exists.
- Look specifically for correctness issues, missing tests, edge cases, accidental broad changes, logging noise, secret exposure, generated-file churn, and formatting problems.
- Fix issues found during self-review, then rerun the relevant verification.
- Post a Linear update summarizing verification and any remaining risk.

### 6. Commit, Merge, And Push Main

- Ensure all intended changes are committed. Use a clear commit message that includes the Linear key, such as `ENG-22195: fix checkout retry handling`.
- Re-fetch latest main before integration.
- Prefer a fast-forward-safe integration:
  - Rebase or merge the task branch onto the latest `origin/main`.
  - Re-run targeted verification if the integration changed the code.
  - Push only when the task branch contains the latest `origin/main` as an ancestor.
- Merge into local `main` and push `origin main` when the local checkout is clean and available.
- If local `main` cannot be safely checked out because of unrelated user work, push the verified task branch to `origin main` only when it is a fast-forward of latest `origin/main`, then explain that integration path in Linear and the final response.
- If branch protection, authentication, CI policy, or permissions reject the push, stop and report the exact blocker. Do not bypass protections.

### 7. Close Out

- Update Linear with the final commit hash, whether `origin/main` was pushed, verification commands and results, and any follow-up issues filed.
- Move the Linear issue to the appropriate completed state only when the team's Linear workflow is clear. Otherwise, leave a final comment and let existing automation or humans transition it.
- Final response must include the Linear issue key, worktree path, branch, commit hash, pushed status, verification run, and any follow-up Linear issue IDs.
