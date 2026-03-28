# Site Template

Use these templates when bootstrapping a new design loop project.

## SITE.md Template

```markdown
# Project Vision

> **AGENT INSTRUCTION:** Read this file before every iteration. It is the project's long-term memory.

## 1. Core Identity

| Field | Value |
|-------|-------|
| **Project Name** | [Name] |
| **Mission** | [What the site achieves] |
| **Target Audience** | [Who uses this site] |
| **Voice & Tone** | [Personality descriptors — warm, professional, playful, etc.] |
| **Region** | [Australia / US / UK — affects spelling, phone format, imagery] |

## 2. Visual Language

Reference these when writing baton prompts.

- **Primary Vibe**: [Main aesthetic — e.g. "Clean and modern"]
- **Secondary Vibe**: [Supporting aesthetic — e.g. "Warm and approachable"]
- **Anti-Vibes**: [What to avoid — e.g. "Not corporate, not cluttered"]

## 3. Technical Setup

- **Output Directory**: `site/public/`
- **CSS**: Tailwind CSS via CDN (no build step)
- **Dark Mode**: [Yes/No] — if yes, via class toggle
- **Fonts**: [Google Fonts import URL]

## 4. Live Sitemap

Update this when a page is successfully generated.

- [x] `index.html` — Homepage with hero, features, CTA
- [ ] `about.html` — Company story and team
- [ ] `services.html` — Service offerings with pricing
- [ ] `contact.html` — Contact form and location map

## 5. Roadmap (Backlog)

Pick the next task from here. Remove items as they're completed.

### High Priority
- [ ] Build about page with team section
- [ ] Build services page with pricing cards

### Medium Priority
- [ ] Build contact page with form
- [ ] Build FAQ page

### Low Priority
- [ ] Blog index page
- [ ] Individual blog post template

## 6. Creative Freedom

When the roadmap is empty, follow these guidelines to add pages:

1. **Stay on-brand** — new pages must fit the established vibe
2. **Enhance the core** — support the site mission
3. **Naming convention** — lowercase, descriptive filenames (e.g. `team.html`)

### Ideas to Explore
- [ ] `testimonials.html` — Customer reviews and case studies
- [ ] `gallery.html` — Project portfolio with image grid
- [ ] `faq.html` — Frequently asked questions with accordion

## 7. Rules of Engagement

1. Do NOT recreate pages already marked `[x]` in Section 4
2. ALWAYS update `.design/next-prompt.md` before completing an iteration
3. Remove consumed ideas from Section 6
4. Copy header/nav/footer from existing pages — never regenerate
5. All internal links must point to real pages
```

## DESIGN.md Template

Generate this using the `design-system` skill, or create manually:

```markdown
# Design System: [Project Name]

## 1. Visual Theme & Atmosphere

[Describe the mood, density, and aesthetic philosophy. Use evocative language.]

Example: "Airy and modern with generous whitespace. Warm undertones soften the
minimal layout. Typography does the heavy lifting — large, confident headings
with understated body text."

## 2. Colour Palette & Roles

| Role | Name | Value | Usage |
|------|------|-------|-------|
| Primary | [Descriptive Name] | `#hexcode` | Buttons, links, active states |
| Primary Foreground | [Name] | `#hexcode` | Text on primary backgrounds |
| Secondary | [Name] | `#hexcode` | Supporting elements, badges |
| Background | [Name] | `#hexcode` | Page background |
| Surface | [Name] | `#hexcode` | Cards, containers |
| Text Primary | [Name] | `#hexcode` | Headings, body text |
| Text Secondary | [Name] | `#hexcode` | Captions, metadata |
| Border | [Name] | `#hexcode` | Dividers, input borders |
| Accent | [Name] | `#hexcode` | Highlights, notifications |

### Dark Mode (if applicable)

| Role | Light Value | Dark Value |
|------|-------------|------------|
| Background | `#hexcode` | `#hexcode` |
| Surface | `#hexcode` | `#hexcode` |
| Text Primary | `#hexcode` | `#hexcode` |

## 3. Typography

| Element | Font | Weight | Size | Line Height |
|---------|------|--------|------|-------------|
| H1 | [Font Family] | 700 | 3rem / 48px | 1.1 |
| H2 | [Font Family] | 600 | 2rem / 32px | 1.2 |
| H3 | [Font Family] | 600 | 1.5rem / 24px | 1.3 |
| Body | [Font Family] | 400 | 1rem / 16px | 1.6 |
| Small | [Font Family] | 400 | 0.875rem / 14px | 1.5 |

Google Fonts import:
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

## 4. Component Styles

### Buttons
- **Primary**: [Background colour], [text colour], [border-radius], [padding]
- **Secondary**: [Outline style], [hover behaviour]
- **Hover**: [Transition description — e.g. "darken 10%, subtle lift shadow"]

### Cards
- **Background**: [Surface colour]
- **Border**: [1px border-colour or none]
- **Border Radius**: [e.g. 12px / rounded-xl]
- **Shadow**: [e.g. "whisper-soft diffused shadow" or "none"]
- **Padding**: [e.g. 1.5rem]

### Navigation
- **Style**: [Sticky/static], [background treatment]
- **Active indicator**: [Underline, background, colour change]
- **Mobile**: [Hamburger menu, slide-out drawer, bottom nav]

### Forms
- **Input style**: [Border, background, border-radius, focus ring]
- **Labels**: [Position, weight, colour]
- **Validation**: [Error colour, success colour, message placement]

## 5. Layout Principles

- **Max content width**: [e.g. 1200px / max-w-7xl]
- **Section padding**: [e.g. py-16 md:py-24]
- **Grid**: [e.g. 12-column, gap-8]
- **Whitespace philosophy**: [Generous / compact / balanced]

## 6. Design System Notes for Generation

**Copy this entire block into every baton prompt:**

**DESIGN SYSTEM (REQUIRED):**
- Platform: Web, Desktop-first, responsive
- Theme: [Light/Dark], [descriptors]
- Background: [Description] (#hex)
- Surface: [Description] (#hex)
- Primary: [Description] (#hex) for [role]
- Text: [Description] (#hex)
- Font: [Font name] via Google Fonts
- Corners: [Description — e.g. "Softly rounded, 12px"]
- Shadows: [Description — e.g. "Whisper-soft diffused shadows"]
- Spacing: [Description — e.g. "Generous whitespace, py-16 sections"]
```
