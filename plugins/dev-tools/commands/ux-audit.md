---
description: Run a UX walkthrough and QA sweep on a live web app
argument-hint: "[quick: spot check | standard: walkthrough | thorough: overnight | exhaustive: every element]"
allowed-tools: "*"
---

Load the `ux-audit` skill and run the audit.

Parse $ARGUMENTS for depth:
- `quick` — 5-10 min spot check, one user flow
- `standard` — 20-40 min walkthrough + QA sweep (default)
- `thorough` — 1-3 hours, multiple personas, all pages, scenario tests, overnight mode
- `exhaustive` — 4-8+ hours, every interactive element on every page clicked/tested/screenshotted

URL is auto-detected from wrangler.jsonc or running dev server.

Examples: `/ux-audit exhaustive`, `/ux-audit thorough`, `/ux-audit quick`
