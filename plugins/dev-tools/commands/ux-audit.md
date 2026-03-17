---
description: Run a UX walkthrough and QA sweep on a live web app
argument-hint: "[quick|standard|thorough] [url]"
allowed-tools: "*"
---

Load the `ux-audit` skill and run the audit.

Parse $ARGUMENTS for:
- **Depth**: `quick` (5-10 min spot check), `standard` (20-40 min, default), or `thorough` (1-3 hours, autonomous overnight mode)
- **URL**: Starting point for the app. If not provided, check for a running dev server or ask.

Examples: `/ux-audit thorough http://localhost:5173`, `/ux-audit quick`, `/ux-audit`
