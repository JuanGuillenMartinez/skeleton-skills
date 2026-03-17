---
name: skeleton-bootstrapping
description: >
  Use at the start of every conversation. Loads CLAUDE.md + ARCHITECTURE.md
  for the target repo and routes intent to the appropriate skeleton:* skill.
---

# Skeleton Bootstrapping

Entry point for every session. Loads context and routes intent.

**Announce:** "Using skeleton:bootstrapping to load project context."

## Checklist

1. Read `./CLAUDE.md` (orchestrator root)
2. Detect target repo from user intent:
   - **Backend:** PHP, Laravel, API, endpoint, module, Action, migration
   - **Frontend:** React, Next.js, component, page, hook, UI, form
   - **Both:** full-stack, full feature, API + UI
   - **Bug:** error, fix, broken, not working
   - **Skill:** create skill, edit skill, workflow
3. Read `{repo}/CLAUDE.md` + `{repo}/ARCHITECTURE.md` for target repo(s)
4. Check for active plans in `docs/plans/`
5. Route to skill:

| Intent | Plan exists? | Destination |
|--------|-------------|-------------|
| New feature | No | `skeleton:designing` |
| New feature | Yes, incomplete | `skeleton:laravel-implementing` / `skeleton:nextjs-implementing` |
| Bug / error | N/A | `skeleton:debugging` |
| Create/edit skill | N/A | `skeleton:writing-skills` |

6. Inform user: what context was loaded and where we're routing

## Anti-Rationalizations

| Temptation | Reality |
|------------|---------|
| "I already know this project" | Fresh context window. ARCHITECTURE.md may have changed. Always read it. |
| "It's a trivial change" | Trivial changes in wrong locations are bugs. Architecture defines file paths. |
| "The user told me exactly what to do" | Knowing WHAT isn't knowing WHERE. Load context first. |

## Transitions

| Condition | Destination |
|-----------|-------------|
| New feature, no plan | `skeleton:designing` |
| Existing incomplete plan | Resume implementation |
| Bug / error | `skeleton:debugging` |
| Create / edit skill | `skeleton:writing-skills` |
