# Claude Code Skills

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Production workflow skills for [Claude Code](https://claude.com/claude-code). Each skill guides Claude through a recipe to produce tangible output — scaffolded projects, generated assets, professional documents, deployed services.

11 plugins. 52 skills. Every one produces something.

## Quick Start

```bash
# Add the marketplace (one-time)
/plugin marketplace add jezweb/claude-skills

# Install what you need
/plugin install cloudflare@jezweb-skills
/plugin install writing@jezweb-skills
/plugin install dev-tools@jezweb-skills
```

Then just ask Claude what you need — installed skills trigger automatically from natural language.

## Plugins

### Build & Deploy

| Plugin | Skills | What it does |
|--------|--------|-------------|
| **cloudflare** | cloudflare-worker-builder, vite-flare-starter, tanstack-start, hono-api-scaffolder, d1-drizzle-schema, d1-migration, db-seed | Scaffold and deploy Cloudflare Workers, full-stack Vite+React apps, TanStack Start SSR dashboards, Hono APIs, D1/Drizzle schemas, migration workflows, database seeding |
| **shopify** | shopify-setup, shopify-products, shopify-content | Shopify API setup, product creation (single + bulk CSV), content pages, blog posts, SEO metadata |
| **wordpress** | wordpress-setup, wordpress-content, wordpress-elementor | WordPress WP-CLI/REST API access, content management, Elementor page editing |

### Design & Frontend

| Plugin | Skills | What it does |
|--------|--------|-------------|
| **frontend** | tailwind-theme-builder, shadcn-ui, landing-page, product-showcase | Tailwind v4 theming with CSS variables + dark mode, shadcn/ui component installation and customisation, single-file landing pages, app showcase sites from real screenshots |
| **design-assets** | color-palette, favicon-gen, icon-set-generator, image-processing, ai-image-generator | Accessible colour palettes from a single hex, favicon packages, custom SVG icon sets, image resize/convert/optimise, AI image generation (Gemini/GPT) |
| **web-design** | seo-local-business | Local business SEO: JSON-LD schema, meta tags, robots.txt, sitemap.xml |

### Writing & Documents

| Plugin | Skills | What it does |
|--------|--------|-------------|
| **writing** | aussie-business-english, us-business-english, uk-business-english, nz-business-english, resume-cover-letter, proposal-writer, award-application, strategy-document | Regional business English style guides (AU/US/UK/NZ), resumes and cover letters, client proposals, award submissions, SWOT/business plans |
| **social-media** | social-media-posts | Platform-formatted posts for LinkedIn, Facebook, Instagram, Reddit — character limits, hashtag strategies, campaign sequences |

### Developer Tools

| Plugin | Skills | What it does |
|--------|--------|-------------|
| **dev-tools** | project-health, project-docs, app-docs, dev-session, team-update, github-release, brains-trust, git-workflow, ux-audit, responsiveness-check, agent-browser | Project config and permissions, doc generation (ARCHITECTURE/API/DB), app user guides with screenshots, session tracking, team chat updates, GitHub releases, multi-model second opinions, git workflows, UX dogfooding (quick/standard/thorough), responsive layout testing, browser automation |
| **integrations** | gws-setup, gws-install, google-chat-messages, google-apps-script, elevenlabs-agents, mcp-builder, parcel-tracking | Google Workspace MCP setup, Google Chat webhooks, Apps Script automation, ElevenLabs voice agents, MCP server building, Australian parcel tracking via Gmail |
| **knowledge-cortex** | cortex-mine, cortex-query | Mine Gmail history into portable flat files — contacts, clients, communications, knowledge facts |

## All 52 Skills

<details>
<summary>Full skill list with trigger phrases</summary>

| Skill | Triggers on |
|-------|------------|
| `cloudflare-worker-builder` | "scaffold a worker", "new cloudflare project" |
| `vite-flare-starter` | "scaffold a full-stack app", "vite + cloudflare" |
| `tanstack-start` | "tanstack start", "SSR dashboard" |
| `hono-api-scaffolder` | "hono api", "api routes" |
| `d1-drizzle-schema` | "d1 schema", "drizzle schema" |
| `d1-migration` | "d1 migration", "migrate database" |
| `db-seed` | "seed database", "sample data", "demo data" |
| `seo-local-business` | "local seo", "json-ld schema" |
| `tailwind-theme-builder` | "tailwind theme", "set up tailwind" |
| `shadcn-ui` | "shadcn", "install components" |
| `landing-page` | "landing page", "marketing page", "one-page site" |
| `product-showcase` | "showcase site", "product page", "show off the app" |
| `color-palette` | "colour palette", "color palette from hex" |
| `favicon-gen` | "favicon", "generate icons" |
| `icon-set-generator` | "icon set", "svg icons" |
| `image-processing` | "resize image", "convert to webp" |
| `ai-image-generator` | "generate image", "ai image", "hero image" |
| `gws-setup` | "google workspace mcp", "set up gmail mcp" |
| `gws-install` | "install google workspace", "connect gmail" |
| `google-chat-messages` | "google chat webhook", "send chat message" |
| `google-apps-script` | "apps script", "google sheets automation" |
| `elevenlabs-agents` | "elevenlabs agent", "voice agent" |
| `mcp-builder` | "build mcp server", "fastmcp" |
| `parcel-tracking` | "where's my parcel", "track my order", "tracking number" |
| `project-health` | "project health", "audit permissions" |
| `project-docs` | "generate docs", "document architecture", "api docs" |
| `app-docs` | "document the app", "user guide", "screenshot docs" |
| `dev-session` | "start session", "session checkpoint" |
| `team-update` | "team update", "post to chat" |
| `github-release` | "github release", "cut a release" |
| `brains-trust` | "second opinion", "brains trust", "ask gemini" |
| `git-workflow` | "prepare pr", "clean branches" |
| `ux-audit` | "ux audit", "dogfood the app" |
| `responsiveness-check` | "check responsiveness", "test viewports" |
| `agent-browser` | "browser automation", "navigate to" |
| `cortex-mine` | "mine gmail", "extract contacts from email" |
| `cortex-query` | "query cortex", "search mined data" |
| `shopify-setup` | "shopify setup", "connect shopify" |
| `shopify-products` | "shopify products", "add products" |
| `shopify-content` | "shopify pages", "shopify blog" |
| `wordpress-setup` | "wordpress setup", "connect wordpress" |
| `wordpress-content` | "wordpress post", "create page" |
| `wordpress-elementor` | "elementor", "edit elementor page" |
| `social-media-posts` | "social media post", "linkedin post", "instagram caption" |
| `aussie-business-english` | "australian english", "write in aussie style" |
| `us-business-english` | "american english", "us business writing" |
| `uk-business-english` | "british english", "uk business writing" |
| `nz-business-english` | "new zealand english", "nz business writing" |
| `resume-cover-letter` | "write a resume", "cover letter" |
| `proposal-writer` | "write a proposal", "client quote" |
| `award-application` | "award application", "grant submission" |
| `strategy-document` | "swot analysis", "business plan", "okrs" |

</details>

## Philosophy

**Every skill must produce something.** No knowledge dumps — only workflow recipes that create files, projects, or configurations.

**Teach patterns, not ship scripts.** Skills describe what to do; Claude generates scripts adapted to your environment.

## Create Your Own

Use [Anthropic's official skill-creator](https://github.com/anthropics/skills) or ask Claude: "Create a new skill for [use case]"

See [CLAUDE.md](CLAUDE.md) for development conventions.

## History

Started as 105 skills — many were reference guides that Claude's training data made redundant.

- **v1** (tag: `v1-final`) — 105 skills, flat structure
- **v2** — restructured into installable plugins, "every skill must produce something"
- **v12** — 10 plugins, 44 skills
- **v13** (current) — 11 plugins, 52 skills

## License

MIT
