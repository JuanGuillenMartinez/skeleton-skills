---
name: skeleton-finishing
description: >
  Use when implementation is complete and verified, to decide how to
  integrate the work. Handles multi-repo awareness for backend and
  frontend separately (commit/push/PR per repo, backend first).
---

# Skeleton Finishing

Guides the completion of development work with multi-repo awareness.

**Announce:** "Using skeleton:finishing to integrate the completed work."

**Prerequisite:** `skeleton:verifying` passed (all checks green).

## Checklist

1. **Verify prerequisite:** `skeleton:verifying` passed
2. **Inventory affected repos:**
   - `cd backend && git status` — any changes?
   - `cd frontend && git status` — any changes?
   - `git status` (root) — any orchestrator changes (specs, plans)?
3. **Present 4 options:**

   | Option | Description | When to choose |
   |--------|-------------|----------------|
   | **A. Merge local** | Merge branch → main locally | Feature complete, total confidence |
   | **B. Create PR** | Push branch + create PR with `gh` | Needs review from another human |
   | **C. Keep as-is** | Leave branch without merge | WIP, will continue in another session |
   | **D. Discard** | Delete branch (**requires text confirmation**) | Failed experiment, disposable prototype |

4. **For full-stack features (2 repos):**
   - Backend commit/push/PR **FIRST**
   - Frontend commit/push/PR **SECOND**
   - Separate PRs: frontend PR references the backend PR
   - Format:
     ```
     Backend PR: feat(mod): [description]
     Frontend PR: feat(mod): [description] — depends on backend#XX
     ```
5. **For orchestrator changes** (specs, plans):
   - Commit to root repo separately
6. **Execute chosen option:**
   - Option A: `git checkout main && git merge {branch}`
   - Option B: `git push -u origin {branch} && gh pr create ...`
   - Option C: Report branch name and status
   - Option D: Text confirmation → `git branch -D {branch}`
7. **Cleanup** worktree if applicable (options A, B, D only)

## Correct Example

```
Full-stack feature completed:

1. ✅ Verification passed
2. ✅ backend has 3 commits, frontend has 2 commits
3. User chooses: "B. Create PR"
4. ✅ Backend: git push -u origin feat/sales-cancel
   ✅ Backend: gh pr create --title "feat(sales): add cancellation endpoint"
   → PR #42 created
5. ✅ Frontend: git push -u origin feat/sales-cancel-ui
   ✅ Frontend: gh pr create --title "feat(sales): add cancel UI — depends on backend#42"
   → PR #18 created
6. ✅ Root: git add docs/plans/ && git commit && git push
```

## Incorrect Example

```
1. ❌ Agent pushes frontend PR first (backend API doesn't exist yet in main)
2. ❌ Agent creates single PR across both repos (they're separate repos)
3. ❌ Agent merges without verification passing
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Any option completed | End of workflow |
| Tests fail during merge | `skeleton:verifying` (go back) |
| Discard without confirmation | BLOCKED — require text confirmation |
