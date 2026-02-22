---
name: playwright-ui-testing
description: >
  Generate Playwright test suites that verify UI components are visible,
  well-organised, and appear at the right time on the right page. Produces
  runnable .spec.ts files with assertions for layout, component presence,
  visual hierarchy, and conversion optimisation. Trigger with 'playwright
  tests', 'ui test suite', 'component visibility tests', 'conversion
  tests', 'page component tests', or 'generate playwright tests'.
compatibility: claude-code-only
---

# Playwright UI Testing

Generate runnable Playwright test suites that verify front-end components are present, visible, correctly organised, and conversion-optimised. Unlike manual UX audits, this skill produces **actual test files** that run in CI and catch regressions.

**Output**: `tests/ui/*.spec.ts` files + `playwright.config.ts` if missing.

## Prerequisites

Before generating tests, verify the project has Playwright:

```bash
# Check for existing Playwright installation
ls node_modules/@playwright/test 2>/dev/null || ls node_modules/playwright 2>/dev/null
```

If missing, confirm with the user then install:

```bash
npm init playwright@latest
```

If the project already has a `playwright.config.ts`, respect its settings. Only create one from the template in [assets/playwright.config.template.ts](assets/playwright.config.template.ts) if none exists.

## Workflow

### Step 1: Discover Routes and Pages

Identify all pages to test. Check (in priority order):

1. **Router config** — React Router, Vue Router, Next.js pages/app dir, SvelteKit routes
2. **Sitemap** — `sitemap.xml` or sitemap generation config
3. **Navigation components** — sidebar, header nav, footer links
4. **User input** — ask which pages/flows matter most

Build a page inventory:

```
/ (homepage)
/about
/pricing
/contact
/dashboard (auth required)
/dashboard/settings
```

### Step 2: Scan Each Page

For each page, use browser automation to capture what's on screen. Use whichever tool is available:

1. **Playwright MCP** (`mcp__plugin_playwright_playwright__*`) — navigate and snapshot
2. **Chrome MCP** (`mcp__claude-in-chrome__*`) — read page content from real browser
3. **playwright-cli** — `open`, `snapshot`, `screenshot`

At each page, identify:

- **Primary components** — hero, nav, footer, sidebar, main content area
- **Interactive elements** — buttons, forms, links, dropdowns, modals
- **Content blocks** — headings, text sections, images, cards, lists
- **Conversion elements** — CTAs, pricing tables, testimonials, trust signals, forms
- **Loading/timing** — skeleton states, lazy-loaded content, animations

### Step 3: Generate Test Files

Create one test file per page or feature area in `tests/ui/`:

```
tests/ui/
├── homepage.spec.ts
├── pricing.spec.ts
├── contact.spec.ts
├── navigation.spec.ts       # Cross-page nav tests
└── conversion-elements.spec.ts  # CRO-specific tests
```

See [references/test-patterns.md](references/test-patterns.md) for Playwright assertion patterns covering:

- Component visibility (`toBeVisible()`)
- Element ordering and layout (`boundingBox()` comparisons)
- Viewport-specific rendering (mobile vs desktop)
- Loading sequence (wait for selectors, network idle)
- Interaction flows (click → expect result)

### Step 4: Generate Conversion Tests

For marketing/product pages, generate tests against the conversion checklist. Read [references/conversion-checklist.md](references/conversion-checklist.md) for the full CRO criteria.

Key areas:

| Category | What to Assert |
|----------|----------------|
| **Above the fold** | Primary CTA visible without scrolling |
| **Value proposition** | Headline + subheadline present and prominent |
| **Trust signals** | Logos, testimonials, or social proof visible |
| **Form friction** | Field count, labels, button text, error handling |
| **Visual hierarchy** | CTA stands out from surrounding content |
| **Mobile CRO** | Sticky CTA, thumb-friendly targets, no hidden CTAs |

### Step 5: Run and Report

```bash
npx playwright test tests/ui/
```

If tests fail, categorise failures:

| Failure Type | Action |
|-------------|--------|
| Element missing | Component not rendered — likely a bug or routing issue |
| Element not visible | CSS/z-index/overflow issue — may need layout fix |
| Wrong position | Layout regression — component moved from expected location |
| Timeout | Slow load or element never appears — performance or logic issue |
| Mobile-only fail | Responsive breakpoint issue |

Report results to the user with specific file:line references and fix suggestions.

## Operating Modes

### Mode 1: Full UI Test Suite

**When**: "generate playwright tests", "ui test suite", "test all pages"

Run the full workflow (Steps 1-5). Produces a complete test suite covering every discovered page.

### Mode 2: Component Visibility Tests

**When**: "component tests for [page]", "check components on [page]", "visibility tests"

Focused on a single page or set of pages. Generates tests asserting that expected components are present and visible in the correct order.

### Mode 3: Conversion Audit Tests

**When**: "conversion tests", "CRO tests", "test conversion elements", "landing page tests"

Generates tests specifically for conversion optimisation elements. Read [references/conversion-checklist.md](references/conversion-checklist.md) before generating.

### Mode 4: Page Flow Tests

**When**: "test user flow", "journey tests", "flow from [page] to [page]"

Tests multi-page user journeys — clicking a CTA leads to the right page, completing a form shows success, navigation works end-to-end.

## Test Naming Conventions

Use descriptive test names that read as specifications:

```typescript
test.describe('Homepage', () => {
  test('displays hero section with headline and primary CTA', async ({ page }) => {});
  test('shows trust signals below the fold', async ({ page }) => {});
  test('primary CTA navigates to signup page', async ({ page }) => {});
  test('renders correctly on mobile viewport', async ({ page }) => {});
});
```

## Autonomy Rules

- **Just do it**: Read project files, scan route configs, generate test files, run tests
- **Brief confirmation**: Before installing Playwright, before creating playwright.config.ts
- **Ask first**: Before modifying existing test files, before changing project configuration

## Reference Files

| When | Read |
|------|------|
| Writing test assertions | [references/test-patterns.md](references/test-patterns.md) |
| Generating conversion tests | [references/conversion-checklist.md](references/conversion-checklist.md) |
| Setting up Playwright config | [references/config-guide.md](references/config-guide.md) |
