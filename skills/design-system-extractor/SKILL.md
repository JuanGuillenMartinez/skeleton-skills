---
name: design-system-extractor
description: Examines a /design folder in the project to extract the visual identity and generate a construction manual (DESIGN_SYSTEM.md) that enables creating new views from scratch with the same style and technical stack. Use this skill when the user wants to analyze mockups, HTML views, or screenshots to extract visual patterns, create style guides, document a design system, or unify visual identity. Also when they mention /design folders, screen.png, mockups, or say "review my designs", "follow this style", "make me a view like this", or "create a new screen for my project". It's not just documentation — it's the recipe to replicate the design.
---

# Design System Extractor

Generates a `DESIGN_SYSTEM.md` by analyzing the project's `/design` folder (each module has `code.html` + `screen.png`). The result is a construction manual that enables creating new views identical in style and stack.

## Guiding principle: Stack fidelity

New views use EXACTLY the same technologies, libraries, CDNs (same URLs and versions), fonts, and icons as the project. Nothing new is introduced and nothing existing is replaced. The DESIGN_SYSTEM.md documents a **mandatory stack** and a list of **prohibited technologies**.

## Workflow

### Step 1: Discovery

```bash
find /design -type f \( -name "code.html" -o -name "screen.png" \) | sort
```
Inform the user how many modules exist and which ones. If the structure varies, adapt.

### Step 2: Analysis of each module

**From the code (`code.html`) extract with exact values:**

1. **Technical stack** — Every `<link>` and `<script>` from the `<head>` (literal URLs), complete `tailwind.config` if applicable, Google Fonts with weights, icon library with parameters, global styles in `<style>`. All of this is the mandatory stack that must be copied identically.

2. **Tokens** — Colors (semantic name + hex + role), typography (family + weight + size per element), spacing (scale used), borders (thickness, color, radius per component), shadows (exact values per intensity), iconography (library, style, size).

3. **Components** — For each one (buttons, cards, inputs, nav, footer, badges, etc.): HTML structure with exact classes, variants, interactive states (hover/focus/active/disabled), copyable functional snippet.

4. **Sections** — Each section type (hero, products, testimonials, features, footer): layout (grid/flex, columns), internal composition (which components and in what order), decoration (backgrounds with opacity, orbs, separators).

5. **General layout** — Page structure, max width, responsive paddings, navigation pattern.

**From the screenshot (`screen.png`) complement:**
Personality/visual tone, hierarchy (what draws attention first), information density, decorative details that give character. **If code and image differ, the image takes precedence.**

### Step 3: Synthesis (if multiple modules exist)

Identify consistent patterns (= the system), inconsistencies (point out which should be the standard), and distill the visual philosophy into 2-3 concrete phrases with values (not "modern design" but rather "turquoise-coral-mustard palette, rounded-2xl/3xl corners, Montserrat bold for headings + Lexend for body").

### Step 4: Generate DESIGN_SYSTEM.md

Document structure:

```
# Design System — [Name]

> [Visual philosophy: 2-3 phrases with concrete values]

## 1. Mandatory stack

### Dependencies
| Type | Name | Exact URL |
[Table with ALL dependencies]

### Complete <head> block (copy as-is to each new view)
```html
[Full project <head>]
```

### Tailwind config (or equivalent)
```javascript
[Complete config]
```

### Base styles
```css
[Global styles]
```

### Prohibited technologies
[List of what must NOT be introduced: other frameworks, other fonts, other icons, etc.]

## 2. Design tokens

### Colors
| Name | Class/variable | Hex | Usage |

### Typography
| Role | Family | Weight | Size | When to use |

### Spacing, borders, shadows, iconography
[Tables with exact values]

## 3. Component blueprints

For each component:
- Copyable HTML snippet with real classes
- Variants and states
- Usage notes

[Buttons, Cards, Header, Footer, Badges, Forms, etc.]

## 4. Section blueprints

For each section type:
- Layout and composition
- Which components it uses
- HTML skeleton with comments

[Hero, Products, Testimonials, Features, Footer, etc.]

## 5. Recipe for a new view

1. Copy the exact <head> boilerplate
2. Verify stack — only project technologies
3. Build skeleton: Header → Sections → Footer
4. Choose sections from blueprints
5. Fill in with components from blueprints
6. Use only documented tokens
7. Review against golden rules

## 6. Golden rules

[5-10 ultra-specific rules, each with:]
- What to do (exact values / classes)
- What to avoid

## 7. Anti-patterns

### Stack violations (most severe)
[Which technologies not to introduce and why]

### Style violations
[Which visual decisions to avoid]
```

### Step 5: Validation

Ask the user: Does it reflect the visual identity? Is any component missing? Adjust any rules?

### Agent-Consumable Output

The generated DESIGN_SYSTEM.md MUST include these sections so `skeleton:nextjs-implementing` can consume them:

1. **Color Tokens** — Tailwind class mapping (e.g., primary → `bg-blue-600`)
2. **Typography Scale** — sizes, weights, line heights as Tailwind classes
3. **Spacing Scale** — standard spacing values
4. **Component Library** — shadcn/ui components used + custom components
5. **Layout Patterns** — dashboard, form, list/table layouts
6. **Status/Badge Variants** — active/inactive/error display
7. **Form Patterns** — field layout, error display, button placement

## Quality criteria

The document is complete if:
1. A new dev can create a view using only this document
2. The snippets work when copied and pasted with the boilerplate
3. 3 devs would create views that look like they belong to the same project
4. There are no ambiguous phrases without exact values
5. It is clear which technologies are used and which are prohibited

## Notes

- With a single module, use it as the single source of truth.
- If there are inconsistencies between modules, suggest the most complete/frequent standard.
- Save where the user indicates, or alongside /design by default.
- If functionality not covered by the stack is needed, solve with vanilla CSS/JS or whatever the project already has — never add dependencies without authorization.
