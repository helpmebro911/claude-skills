---
description: Mine Gmail history into a local knowledge base
argument-hint: "[dry-run|from DATE]"
---

Load the `cortex-mine` skill.

Parse $ARGUMENTS:
- `dry-run` — preview what would be extracted without saving
- `from YYYY-MM-DD` — only process emails from this date

Default: incremental mine from last run.
