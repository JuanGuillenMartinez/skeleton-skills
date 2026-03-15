# skeleton-skills

Claude Code plugin for full-stack Laravel 12 + Next.js 15 modular monolith projects.

## Install

```bash
claude plugins add skeleton-skills@skeleton-skills
```

Or add to your project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "skeleton-skills@skeleton-skills": true
  }
}
```

## Skills

| Skill | When |
|-------|------|
| `skeleton-bootstrapping` | Start of every session — loads context, routes intent |
| `skeleton-brainstorming` | Before any creative work — explores requirements |
| `skeleton-planning` | After brainstorming — creates atomic TDD task plan |
| `skeleton-implementing` | Routes plan tasks to stack-specific skills |
| `skeleton-laravel-implementing` | TDD for backend tasks (Pest, Pint, Modules/) |
| `skeleton-nextjs-implementing` | TDD for frontend tasks (Vitest, MSW, modules/) |
| `skeleton-architecture-guarding` | Post-task validation (pint, phpstan, lint, typecheck) |
| `skeleton-debugging` | Systematic 4-phase debugging |
| `skeleton-verifying` | Final verification before claiming done |
| `skeleton-finishing` | Merge/PR/keep/discard — multi-repo aware |
| `skeleton-writing-skills` | Meta: create/edit skills with RED-GREEN-REFACTOR |
| `design-system-extractor` | Analyze /design folder, generate DESIGN_SYSTEM.md |

## Workflow

```
brainstorm → plan → implement (TDD) → guard → verify → finish
```

- **Backend first, frontend second** for full-stack features
- **TDD obligatorio** — test first, implement second
- **Architecture guards** before every merge

## License

MIT
