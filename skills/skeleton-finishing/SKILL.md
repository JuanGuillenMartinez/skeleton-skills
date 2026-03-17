---
name: skeleton-finishing
description: >
  Use when implementation is complete and verified. Handles multi-repo
  commit/push/PR with backend-first ordering.
---

# Skeleton Finishing

Guides completion of development work with multi-repo awareness.

**Announce:** "Using skeleton:finishing to integrate the completed work."

**Prerequisite:** `skeleton:validating final` passed.

## Checklist

1. **Inventory changes:**
   - `cd backend && git status`
   - `cd frontend && git status`
   - `git status` (root — specs, plans)
2. **Present options:**

   | Option | When |
   |--------|------|
   | **A. Merge local** | Feature complete, total confidence |
   | **B. Create PR** | Needs review from another human |
   | **C. Keep as-is** | WIP, will continue later |
   | **D. Discard** | Failed experiment (requires text confirmation) |

3. **Execute** (backend FIRST for full-stack):
   - **A:** `git checkout main && git merge {branch}`
   - **B:** `git push -u origin {branch} && gh pr create ...`
   - **C:** Report branch name
   - **D:** Confirm → `git branch -D {branch}`
4. **Full-stack PRs:** Separate per repo. Frontend PR references backend PR.

## Transitions

| Condition | Destination |
|-----------|-------------|
| Option completed | End of workflow |
| Tests fail during merge | Back to `skeleton:validating` |
