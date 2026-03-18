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
| `skeleton-designing` | Before any new feature — explores requirements + creates TDD plan |
| `skeleton-laravel-implementing` | TDD for backend tasks (Pest, Pint, Modules/) |
| `skeleton-nextjs-implementing` | TDD for frontend tasks (Vitest, MSW, modules/) |
| `skeleton-validating` | After each task and as final check before completion |
| `skeleton-finishing` | Merge/PR/keep/discard — multi-repo aware |
| `skeleton-debugging` | Systematic 4-phase debugging |
| `skeleton-writing-skills` | Meta: create/edit skills with RED-GREEN-REFACTOR |
| `design-system-extractor` | Analyze /design folder, generate DESIGN_SYSTEM.md |

## Workflow

```
bootstrap → design → implement (TDD) → validate → finish
```

- **Backend first, frontend second** for full-stack features
- **TDD mandatory** — test first, implement second
- **Architecture guards** after every task and before completion

## License

MIT
