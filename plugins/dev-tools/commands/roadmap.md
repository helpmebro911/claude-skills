---
description: Generate a comprehensive phased delivery plan for building an entire application
argument-hint: "[product brief or topic]"
---

Load the `roadmap` skill.

If $ARGUMENTS contains a product description, use it as the starting brief. If a deep-research brief exists in `.jez/artifacts/research-brief-*.md`, offer to use it as input.

Produces a `docs/ROADMAP.md` detailed enough that "build phase N" is a complete instruction for Claude Code.

Examples: `/roadmap markdown note app on cloudflare`, `/roadmap` (asks for brief), `/roadmap from deep-research brief`
