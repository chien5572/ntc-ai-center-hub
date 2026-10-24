---
name: ntchub-git-safety
description: >-
  Use this skill when implementing or modifying the Git/GitHub service (`ARCH §9`, `SPEC US-M1-11`),
  git worktree management, branch push guards, Pull Request creation/merge conditions,
  branch cleanup (`D-16`), or repository creation (`US-M3-01`).
---

# NTC Hub Git & GitHub Safety Service (🛡 Anti-Goal #1)

Enforces the supreme anti-goal of `ntc-ai-center-hub`: **Never push broken code or corrupt a repository** (PRD §2.3 #1, §4.6; ARCH §9; SPEC US-M1-11 🛡, US-M3-01 🛡).

## When to Activate
- Implementing `GitService` and `GitHubService` in `hubd`.
- Managing mirror clones (`~/.ntc-hub/repos/<slug>/`) and task worktrees (`~/.ntc-hub/work/<id>/`).
- Implementing commit, push, PR creation, PR merge, rebase/update-from-base, or branch cleanup.

## Hardcoded Safety Guards (🛡 Must Be Enforced in Code, Never Configurable)

### 1. Worktree & Branch Naming Invariant (FR-27, D-11, AC-M1-11.1)
- Every task operates in an isolated git worktree at `~/.ntc-hub/work/<task_id>/` on branch `hub/<task_id>-<slug>`.
- The worktree's `origin` remote inside the directory must **NOT** embed credentials (`https://github.com/<owner>/<repo>.git`), so agents inside the container cannot push (`AC-M1-05.3 🛡`).
- Only `hubd` (running outside the agent container) supplies GitHub App installation tokens / PAT when executing verified `git push` or GitHub API calls.

### 2. Hard Ref & Force-Push Guards (🛡 AC-M1-11.2)
Before executing any push or branch deletion in `GitService`, validate in code:
- **Target Ref Guard**: `ref` MUST match `^refs/heads/hub/[0-9]+-[a-z0-9-]+$`. Reject `main`, `master`, `develop`, `refs/tags/*`, or any non-`hub/*` ref with a `GitSafetyViolationError`.
- **Force Push Guard**: Reject `--force`, `-f`, or `+refs/...` unconditionally. The **only** permitted lease push is `--force-with-lease` onto the task's own `refs/heads/hub/<task_id>-<slug>` when `hubd` itself performs a rebase onto the updated default branch.
- **Repo Settings Guard**: Never modify repository settings except enabling branch protection on the default branch of a newly created repository (`US-M3-01`).
- **Sanitize Worktree Before Push (`AC-M1-11.6 🛡`)**: Before `hubd` commits/pushes after an agent stage, verify `.git/config` and `.git/hooks` were not tampered with by the agent.

### 3. Four-Condition PR Merge Gate (🛡 FR-29, AC-M1-11.3, AC-M1-11.4, D-15)
`hubd` may ONLY merge a task into the default branch via the GitHub Pull Request Merge API, and ONLY when **ALL 4** conditions hold simultaneously:
1. **Hub Independent Test Pass**: `hubd` ran `project.test_cmd` on the **current HEAD SHA** of the PR branch and it exited `0`. (If `project.test_cmd` is empty, auto-merge is permanently forbidden — `AC-M1-08.5 🛡`).
2. **Structured Review Approved**: Latest `review.json` passes schema + consistency check (`D-07`) with `verdict == "approve"`. (If `test` or `review` stage was disabled in pipeline, auto-merge is forbidden — `D-15`).
3. **Merge Gate Cleared**: Either the `merge` approval gate was explicitly approved by the owner, OR `merge` gate is disabled / `/auto on` (subject to conditions 1 & 2).
4. **Up-to-Date & Conflict-Free**: If `default_branch` advanced, `hubd` updates/rebases `hub/<id>-<slug>` from base and **re-runs `project.test_cmd`** before merging (`AC-M1-11.4`). If conflicts occur, transition task to `queued` for a `fix` stage.

### 4. Branch Lifecycle & Audit Log (D-16, AC-M1-11.5, AC-M1-11.7)
- **On Successful Merge**: Immediately delete remote branch `hub/<id>-<slug>` and log to `audit_log`.
- **On `cancelled` or `failed`**: Keep remote branch `hub/<id>-<slug>` until worktree TTL cleanup (default 7 days) or explicit `/cleanup <id>` / `hub cleanup <id>`.
- **Cleanup Guard**: Refuse to delete any branch not matching `hub/<id>-<slug>` created by `hubd`.
