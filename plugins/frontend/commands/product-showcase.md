---
description: Generate a marketing site from a running app with real screenshots
argument-hint: "[quick: hero + features | standard: full showcase | thorough: every screen] [url]"
---

Load the `product-showcase` skill.

Parse $ARGUMENTS for:
- **Depth**: `quick` (hero + 3-4 feature screenshots, single page), `standard` (full showcase with how-it-works, feature grid, CTAs), `thorough` (every screen captured, comprehensive multi-section site). Default: standard
- **URL**: app to showcase. If not provided, check wrangler.jsonc for deployed URL, then running dev server, then ask.

Examples: `/product-showcase thorough https://app.example.com`, `/product-showcase quick`, `/product-showcase`
