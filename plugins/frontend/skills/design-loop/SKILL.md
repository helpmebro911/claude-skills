---
name: design-loop
description: "Autonomous multi-page site builder using a baton-passing loop pattern. Each iteration reads a task from .design/next-prompt.md, generates a page with Claude's HTML/CSS/Tailwind, integrates it into the site, verifies visually via browser automation, then writes the next task to keep the loop going. Drives complete website builds from a single starting prompt. Triggers: 'design loop', 'build the site', 'build all pages', 'autonomous site build', 'baton loop', 'next page', 'keep building pages'."
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
compatibility: claude-code-only
---

# Design Loop — Autonomous Site Builder

Build complete multi-page websites through an autonomous loop. Each iteration reads a task, generates a page, integrates it, verifies it visually, then writes the next task to keep going.

## Overview

The Design Loop uses a "baton" pattern — a file (`.design/next-prompt.md`) acts as a relay baton between iterations. Each cycle:

1. Reads the current task from the baton
2. Generates the page (via Claude or Google Stitch)
3. Integrates into the site structure (navigation, links)
4. Verifies visually via browser automation (if available)
5. Updates site documentation
6. Writes the NEXT task to the baton — keeping the loop alive

This is orchestration-agnostic. The loop can be driven by:
- **Human-in-loop**: User reviews each page, then says "next" or "keep going"
- **Fully autonomous**: Claude runs continuously until the site is complete
- **CI/CD**: Triggered on `.design/next-prompt.md` changes

## Generation Backends

The loop supports two generation backends:

| Backend | Setup | Quality | Speed | Best for |
|---------|-------|---------|-------|----------|
| **Claude** (default) | Zero dependencies | Great — production-ready HTML/Tailwind | Fast | Most projects, full code control |
| **Google Stitch** | `npm install @google/stitch-sdk` + API key | Higher fidelity AI designs | ~10-20s/screen | Design-heavy projects, visual polish |

### Detecting Stitch

At the start of each loop, check if Stitch is available:

1. Check if `@google/stitch-sdk` is installed: `ls node_modules/@google/stitch-sdk 2>/dev/null`
2. Check if `STITCH_API_KEY` is set in `.dev.vars` or environment
3. Check if `.design/metadata.json` exists (contains Stitch project ID)

If all three are present, use Stitch. Otherwise, fall back to Claude generation.

### Using Stitch SDK

See `references/stitch-sdk.md` for the full SDK reference. Quick usage:

```typescript
import { stitch } from "@google/stitch-sdk";

// Create or reference a project
const project = stitch.project(projectId);

// Generate a screen from the baton prompt
const screen = await project.generate(batonPrompt, "DESKTOP");

// Get the HTML and screenshot
const htmlUrl = await screen.getHtml();
const imageUrl = await screen.getImage();

// Download both
// HTML → .design/designs/{page}.html → then process into site/public/{page}.html
// Screenshot → .design/screenshots/{page}.png
```

When using Stitch, the generated HTML may need post-processing:
- Extract and reuse your project's header/nav/footer (Stitch generates standalone pages)
- Ensure Tailwind config matches your DESIGN.md
- Wire internal navigation links

### Stitch Project Persistence

Save Stitch identifiers to `.design/metadata.json` so future iterations can reference them:

```json
{
  "projectId": "4044680601076201931",
  "screens": {
    "index": { "screenId": "d7237c7d78f44befa4f60afb17c818c1" },
    "about": { "screenId": "bf6a3fe5c75348e58cf21fc7a9ddeafb" }
  }
}
```

Use `screen.edit(prompt)` for iterative refinements on existing screens rather than regenerating from scratch.

## Getting Started

### First Run: Bootstrap the Project

If `.design/` doesn't exist yet, create the project scaffolding:

1. **Ask the user** for:
   - Site name and purpose
   - Target audience
   - Desired aesthetic (minimal, bold, warm, etc.)
   - List of pages they want
   - Brand colours (or extract from existing site with `/design-system`)

2. **Create the project files**:

```
project/
├── .design/
│   ├── SITE.md           # Vision, sitemap, roadmap — the project's long-term memory
│   ├── DESIGN.md         # Visual design system — the source of truth for consistency
│   └── next-prompt.md    # The baton — current task with page frontmatter
└── site/
    └── public/           # Production pages live here
```

3. **Write SITE.md** from the template in `references/site-template.md`
4. **Write DESIGN.md** — either manually from user input, or use the `design-system` skill to extract from an existing site
5. **Write the first baton** (`.design/next-prompt.md`) for the homepage

### Subsequent Runs: Read the Baton

If `.design/next-prompt.md` already exists, parse it and continue the loop.

## The Baton File

`.design/next-prompt.md` has YAML frontmatter + a prompt body:

```markdown
---
page: about
layout: standard
---
An about page for Acme Plumbing describing the company's 20-year history in Newcastle.

**DESIGN SYSTEM:**
[Copied from .design/DESIGN.md Section 6]

**Page Structure:**
1. Header with navigation (consistent with index.html)
2. Hero with company photo and tagline
3. Story timeline showing company milestones
4. Team section with photo grid
5. CTA section: "Get a Free Quote"
6. Footer (consistent with index.html)
```

| Field | Required | Purpose |
|-------|----------|---------|
| `page` | Yes | Output filename (without .html) |
| `layout` | No | `standard`, `wide`, `sidebar` — defaults to `standard` |

## Execution Protocol

### Step 1: Read the Baton

```
Read .design/next-prompt.md
Extract: page name, layout, prompt body
```

### Step 2: Consult Context Files

Before generating, read:

| File | What to check |
|------|---------------|
| `.design/SITE.md` | Section 4 (Sitemap) — don't recreate existing pages |
| `.design/DESIGN.md` | Colour palette, typography, component styles |
| Existing pages in `site/public/` | Header/footer/nav patterns to match |

**Critical**: Read the most recent page's HTML to extract the exact header, navigation, and footer markup. New pages must use identical shared elements.

### Step 3: Generate the Page

#### Option A: Claude Generation (Default)

Generate a complete HTML file using Tailwind CSS (via CDN). The page must:

- **Match the design system** from `.design/DESIGN.md` exactly
- **Reuse the same header/nav/footer** from existing pages (copy verbatim)
- **Be self-contained** — single HTML file with Tailwind CDN, no build step
- **Be responsive** — mobile-first, works at all breakpoints
- **Include dark mode** if the design system specifies it
- **Use semantic HTML** — proper heading hierarchy, landmarks, alt text
- **Wire real navigation** — all nav links point to actual pages (existing or planned)

Write the generated file to `site/public/{page}.html`.

#### Option B: Stitch Generation (If Available)

If Stitch SDK is available:

1. Build the prompt by combining the baton body with the DESIGN.md system block
2. Call `project.generate(prompt, deviceType)` to generate the screen
3. Download the HTML from `screen.getHtml()` → save to `.design/designs/{page}.html`
4. Download the screenshot from `screen.getImage()` → save to `.design/screenshots/{page}.png`
5. Post-process the Stitch HTML:
   - Replace the header/nav/footer with your project's shared elements
   - Ensure consistent Tailwind config
   - Wire internal navigation links
6. Save the processed file to `site/public/{page}.html`
7. Update `.design/metadata.json` with the new screen ID

For iterative edits on an existing Stitch screen, use `screen.edit(prompt)` instead of regenerating.

### Step 4: Integrate into the Site

After generating the new page:

1. **Update navigation across ALL existing pages** — add the new page to nav menus
2. **Fix placeholder links** — replace any `href="#"` with real page URLs
3. **Verify cross-page consistency** — header, footer, nav must be identical everywhere
4. **Check internal links** — no broken links between pages

### Step 5: Visual Verification (If Browser Available)

If Playwright CLI or Chrome MCP is available:

1. Start a local server: `npx serve site/public -p 3456`
2. Screenshot the new page at desktop (1280px) and mobile (375px) widths
3. Save screenshots to `.design/screenshots/{page}-desktop.png` and `{page}-mobile.png`
4. Compare visually against the design system
5. Fix any issues (broken layout, wrong colours, inconsistent nav)
6. Stop the server

If no browser automation is available, skip to Step 6.

### Step 6: Update Site Documentation

Edit `.design/SITE.md`:

- Mark the page as complete in Section 4 (Sitemap): `[x] {page}.html — {description}`
- Remove any consumed item from Section 5 (Roadmap) or Section 6 (Ideas)
- Add any new ideas discovered during generation

### Step 7: Write the Next Baton (CRITICAL)

**You MUST update `.design/next-prompt.md` before completing.** This keeps the loop alive.

1. **Choose the next page**:
   - First: Check Section 5 (Roadmap) for pending high-priority items
   - Second: Check Section 5 for medium-priority items
   - Third: Pick from Section 6 (Ideas)
   - Last resort: Invent something that fits the site vision

2. **Write the baton** with:
   - YAML frontmatter (`page`, optional `layout`)
   - Description of the page purpose and content
   - Design system block copied from `.design/DESIGN.md` Section 6
   - Detailed page structure (numbered sections)

3. **If the site is complete** (all roadmap items done, no more ideas):
   - Write a baton with `page: _complete` and a summary of what was built
   - This signals the loop is finished

## Loop Completion

The loop ends when:
- All pages in the roadmap are built (`[x]` in SITE.md Section 4)
- The user says to stop
- The baton contains `page: _complete`

On completion, output a summary:
- Pages built (with links)
- Screenshots (if captured)
- Any remaining ideas for future work

## Cross-Page Consistency Rules

The #1 risk in multi-page generation is **drift** — pages looking slightly different. Prevent this:

| Element | Rule |
|---------|------|
| **Header/Nav** | Copy exact HTML from the most recent page. Never regenerate. |
| **Footer** | Same — copy verbatim, only change active page indicator |
| **Tailwind config** | If using `<script>` config block, it must be identical across pages |
| **Colour values** | Always use the exact hex codes from DESIGN.md, never approximate |
| **Font imports** | Same Google Fonts `<link>` tag across all pages |
| **Spacing scale** | Consistent padding/margin values (document in DESIGN.md) |

## File Structure

```
project/
├── .design/
│   ├── SITE.md              # Project vision, sitemap, roadmap
│   ├── DESIGN.md            # Visual design system (source of truth)
│   ├── next-prompt.md       # The baton — current/next task
│   └── screenshots/         # Visual verification captures
│       ├── index-desktop.png
│       ├── index-mobile.png
│       ├── about-desktop.png
│       └── about-mobile.png
├── site/
│   └── public/              # Production pages
│       ├── index.html
│       ├── about.html
│       ├── services.html
│       └── contact.html
└── .gitignore               # Add .design/screenshots/
```

## Tips

- **Start with the homepage** — it establishes the visual language for everything else
- **Read existing pages before generating** — consistency comes from copying, not recreating
- **One page per iteration** — don't try to generate multiple pages at once
- **Include the design system in every baton** — Claude needs it fresh each time
- **Use the roadmap** — don't generate pages randomly; follow the user's priority order
- **Wire navigation early** — even link to pages that don't exist yet (they will soon)

## Common Pitfalls

- ❌ Forgetting to update `.design/next-prompt.md` (breaks the loop)
- ❌ Recreating a page that already exists in the sitemap
- ❌ Regenerating the header/nav instead of copying from existing pages
- ❌ Not including the design system block in the baton prompt
- ❌ Leaving `href="#"` placeholder links instead of real page URLs
- ❌ Inconsistent Tailwind config across pages
