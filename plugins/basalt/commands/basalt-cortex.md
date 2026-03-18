---
description: Mine knowledge from multiple sources into an Obsidian-compatible vault
argument-hint: "[init|mine|query|stats|sync] [source|search-term|--from DATE]"
---

Load the `basalt-cortex` skill.

Parse $ARGUMENTS for mode and parameters:
- `init` — create ~/.cortex/ vault structure
- `mine [source]` — extract from a source (gmail, chat, slack, drive, local, web). Default: gmail
- `query [term]` — search across all Basalt files
- `stats` — show vault statistics
- `sync` — push to Frond (future)

Examples: `/basalt-cortex mine gmail --from 2026-01-01`, `/basalt-cortex query smith holdings`, `/basalt-cortex init`
