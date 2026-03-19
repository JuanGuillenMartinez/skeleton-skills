---
description: "Use when implementing frontend UI changes. Generates HTML prototype for visual approval before real implementation."
disable-model-invocation: false
---

# skeleton:previewing

Generate an HTML preview prototype for visual approval before implementing in the real frontend stack. This prevents wasting implementation time on layouts the user will reject.

## When to Use

INVOKE this skill when ALL of these are true:
- The task involves creating or significantly changing a UI view/page
- The change is visual (new page, new layout, redesign, new component with complex UI)
- The frontend implementation would take >5 minutes

DO NOT invoke for:
- Bug fixes that don't change layout
- Adding a single field to an existing form
- Backend-only changes
- Changes where the user explicitly said "skip preview" or "just implement it"

## Step 0: Load Visual Context

Before generating the prototype:

1. Read `frontend/DESIGN_SYSTEM.md` from disk (if exists)
   - Extract: color tokens, font families, spacing scale, component patterns
2. Check `/design` folder for relevant mockups/wireframes
3. Check if `anthropics/frontend-design` plugin is active
4. Read the current page/component being modified (if it exists) to maintain consistency

## Step 1: Generate HTML Prototype

Create a SINGLE self-contained HTML file at: `frontend/.previews/{feature-name}.html`

Requirements:
- **Self-contained:** ALL CSS inline or in `<style>`. ALL JS inline in `<script>`. NO external dependencies except CDN links.
- **CDN-powered:** Use Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com">`). Use any icon library via CDN if needed.
- **Mock data:** Hardcode realistic sample data. Never call real APIs.
- **Responsive:** Must look correct at 375px (mobile) and 1440px (desktop) minimum.
- **Interactive enough:** Tabs should switch, dropdowns should open, modals should show/hide. Use vanilla JS.
- **Match design system:** If DESIGN_SYSTEM.md exists, use its exact colors (as Tailwind config in the CDN script), fonts (via Google Fonts CDN), and spacing.
- **Include navigation context:** Show the sidebar/header/layout shell so the user sees the page in context, not floating in isolation.

Structure:
```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Preview: {Feature Name}</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            // Colors from DESIGN_SYSTEM.md
          }
        }
      }
    }
  </script>
  <!-- Google Fonts from DESIGN_SYSTEM.md -->
  <style>
    /* Any custom CSS needed */
  </style>
</head>
<body>
  <!-- Layout shell (sidebar + header) for context -->
  <!-- Main content area with the actual preview -->

  <script>
    // Minimal interactivity (tab switching, modals, dropdowns)
  </script>
</body>
</html>
```

## Step 2: Serve the Preview

Start a local server and provide the URL:

```bash
# Option A: Python (usually available)
cd frontend/.previews && python3 -m http.server 3333 &

# Option B: npx serve (if Node available)
npx serve frontend/.previews -p 3333 &
```

Tell the user:
```
Preview ready: http://localhost:3333/{feature-name}.html

Open in your browser and let me know:
- Does the layout match what you expected?
- Any elements to move, resize, add, or remove?
- Colors, fonts, spacing OK?
- Mobile view OK? (resize browser to check)

I'll iterate on this prototype until you approve, then implement in React/Next.js.
```

## Step 3: Feedback Loop

For EACH round of feedback:

1. Listen to the user's feedback
2. Update the HTML file in place (edit, don't recreate)
3. Tell the user to refresh the browser
4. Ask: "Refresh and check. Anything else to adjust, or is this approved?"

RULES:
- Maximum 5 iteration rounds. If the user is still not satisfied after 5, ask if we should
  rethink the approach entirely.
- Track what changed in each iteration (keep a mental log).
- NEVER say "approved" yourself. Wait for the user to explicitly approve.

## Step 4: Transition to Implementation

When the user approves:

1. Stop the local server: `kill %1` or `lsof -ti:3333 | xargs kill`
2. Save a screenshot note in the plan file:
   ```
   ## Visual Reference
   Approved prototype: frontend/.previews/{feature-name}.html
   Key decisions:
   - [List the visual decisions made during feedback, e.g., "sidebar fixed, not collapsible"]
   - [e.g., "table uses pagination, not infinite scroll"]
   - [e.g., "status badges use colored dots, not text labels"]
   ```
3. Transition to `skeleton:nextjs-implementing` with the prototype as visual reference
4. During implementation, the agent should open the HTML file to verify visual fidelity

The prototype file stays in `.previews/` as reference. Add `frontend/.previews/` to `.gitignore`.

## Step 5: Implementation Fidelity Check

After implementing in React/Next.js:

1. Compare the real implementation visually against the prototype
2. Check: same layout structure, same spacing, same colors, same interactive behavior
3. If there's a significant visual difference, fix it before marking the task as done

RULE: The implemented version should be visually indistinguishable from the approved prototype
(except for real data instead of mock data and real interactivity instead of vanilla JS).
