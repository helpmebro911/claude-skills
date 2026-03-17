---
description: Generate a user guide with screenshots from a running app
argument-hint: "[quick|standard|thorough] [url]"
---

Load the `app-docs` skill.

Parse $ARGUMENTS for:
- **Depth**: `quick` (~10 screenshots), `standard` (~30), `thorough` (~80+). Default: standard
- **URL**: app to document. If not provided, check for a running dev server or ask.
