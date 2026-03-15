---
name: skeleton-writing-skills
description: >
  Use when creating new skeleton:* skills, editing existing proprietary
  skills, or verifying skills work before deployment. Applies RED-GREEN-REFACTOR
  to documentation: baseline without skill, write skill, close loopholes.
---

# Skeleton Writing Skills

Meta-skill for creating and maintaining `skeleton:*` skills. Applies TDD principles to documentation.

**Announce:** "Using skeleton:writing-skills to create/edit this skill."

## Rules

| Rule | Detail |
|------|--------|
| Namespace | Always `skeleton:*`. No exceptions. |
| Frontmatter | Only `name` + `description`. Description = "Use when...", never workflow summary |
| `name` | Max 64 chars, hyphen-case |
| `description` | Max 1024 chars, ONLY trigger conditions |
| Anti-rationalizations | Required for rigid skills. Table: "Excuse" / "Reality". Min 8 entries. |
| Examples | At least 1 correct + 1 incorrect per skill |
| Checklist | Numbered steps the agent MUST complete in order |
| Subagent templates | Separate `.md` files alongside SKILL.md, never inline |
| Flowcharts | Only for non-obvious decisions. NOT for linear instructions |
| References | By name: `skeleton:{name}`. No @ syntax to avoid forced loading |
| Type | Declare explicitly: Rigid or Flexible |
| Third-party policy | NEVER modify existing skills. Create new `skeleton:*` skills instead. |

## Checklist — RED-GREEN-REFACTOR for Documentation

### RED — Baseline (failing test)

1. Define the scenario the skill must cover
2. Execute the scenario WITHOUT the skill (or with current version if editing)
3. Document failures:
   - What went wrong?
   - What was skipped?
   - What file was created in the wrong location?
   - What convention was violated?
4. This is the "failing test" — the baseline against which the skill is measured

### GREEN — Write the skill

5. Create `SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skeleton:{name-in-gerund}
   description: "Use when..."
   ---
   ```
6. Write content:
   - Overview with core principle
   - Checklist with exact steps
   - Anti-rationalizations (if rigid)
   - Correct + Incorrect examples
   - Transitions to other skills
7. Execute the scenario WITH the skill
8. Verify it resolves ALL failures documented in RED

### REFACTOR — Close loopholes

9. Find rationalizations discovered during GREEN
10. Add them to anti-rationalizations table
11. Close documented edge cases
12. Re-execute scenario to verify it still works
13. Verify it doesn't break other existing skills

### Deploy

14. Create directory: `skills/skeleton-{name}/SKILL.md` (or `{repo}/skills/` if stack-specific)
15. Create symlink in `.claude/skills/`:
    ```bash
    cd .claude/skills && ln -sf ../../skills/skeleton-{name} skeleton-{name}
    ```
16. Update `setup.sh` if the skill should persist across clones (setup.sh creates symlinks from gitignored `.claude/skills/`)
17. Verify Claude Code discovers it: read the SKILL.md directly from its path to confirm it's accessible. Note: `.claude/skills/` symlinks are gitignored, so skills may not be discoverable via the `Skill` tool — reading directly is the reliable method.

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "The skill is simple, it doesn't need a test (RED phase)" | Untested skills have issues. Always run the baseline. |
| "I'm just editing text, not code" | Text controls agent behavior. A typo in a trigger can prevent the skill from ever activating. Test it. |
| "I know it works, I wrote it" | Your certainty isn't evidence. Execute the scenario. Always. |
| "There's no way to test a skill automatically" | The test is manual: execute scenario with and without the skill, document the difference. That's the test. |
| "The skill is for future cases, I can't test it now" | If you can't test it now, don't write it now. Skills are written when there's a real scenario. |
| "Anti-rationalizations are unnecessary for this skill" | If you think they're unnecessary, that's the rationalization they're designed to prevent. Add them. |
| "I'll add examples later" | Examples are how the skill is verified. Without them, you can't confirm it works. Add them now. |
| "This skill overlaps with a Superpowers skill" | That's fine — skeleton:* has higher priority. Document the overlap and explain why ours is different. |

## Correct Example

```
Creating skeleton:inventory-patterns skill:

RED:
  Scenario: "Create an Inventory module"
  Without skill: Agent creates App\Models\Inventory (wrong path),
  uses SoftDeletes, puts validation in DTO
  Failures documented: 3 specific violations

GREEN:
  Wrote SKILL.md with module structure rules
  Re-ran scenario: All 3 violations prevented

REFACTOR:
  Discovered: agent still used API Resources instead of DTOs
  Added anti-rationalization for that case
  Re-ran: clean

Deploy:
  Created skills/skeleton-inventory-patterns/SKILL.md
  Symlinked to .claude/skills/
  New session: skill appears in list ✓
```

## Incorrect Example

```
Creating a new skill:

1. ❌ Write SKILL.md directly without testing baseline (no RED)
2. ❌ No anti-rationalizations (rigid skill)
3. ❌ No examples (can't verify behavior)
4. ❌ No symlink (Claude Code can't discover it)
5. ❌ Tested by reading the file (not by executing scenario)
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Skill completed and deployed | End of workflow |
| Skill requires ARCHITECTURE.md changes | Make changes first, then re-test |
| Skill conflicts with existing skill | Investigate, document resolution |
