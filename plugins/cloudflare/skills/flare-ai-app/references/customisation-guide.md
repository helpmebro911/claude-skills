# Flare AI App — Customisation Guide

Complete checklist for turning the starter into a client-specific app.

## Automated (Step 2 in setup)

| Item | File | Default Value |
|------|------|--------------|
| Worker name | `wrangler.jsonc` | `my-ai-app` |
| Database name | `wrangler.jsonc` | `my-ai-app-db` |
| Database ID | `wrangler.jsonc` | `REPLACE_WITH_YOUR_DATABASE_ID` |
| Package name | `package.json` | `vite-flare-ai-starter` |
| Page title | `index.html` | Emoji favicon |

## Domain Configuration (Step 3 — THE KEY STEP)

### config/app.ts — App Identity

| Field | Purpose | Example |
|-------|---------|---------|
| `name` | Display name in sidebar + header | `"Acme Analytics"` |
| `description` | Subtitle / tagline | `"AI-powered business intelligence"` |
| `logoChar` | Single character for sidebar logo | `"A"` |
| `version` | Shown in sidebar footer | `"1.0.0"` |
| `defaultModel` | Initial AI model selection | `"claude-sonnet"` |
| `timezone` | Date/time display | `"Australia/Sydney"` |
| `kbCategories` | KB article category list | `["general", "technical", "faq"]` |
| `features.leadCapture` | Enable demo access gate | `false` |

### config/schema.ts — AI System Prompt

This is the **most important** customisation. The quality of the AI's responses depends entirely on this file.

**Must include:**
- Business context (what the company does, key metrics)
- Table schemas with column descriptions
- Domain terminology and definitions
- Common query patterns (what users typically ask)
- Data quality notes and caveats

**Pattern for table documentation:**
```
### Table: orders
| Column | Type | Description |
|--------|------|-------------|
| id | INTEGER | Auto-increment primary key |
| customer_name | TEXT | Full customer name |
| total | REAL | Order total in AUD (GST-inclusive) |
| status | TEXT | One of: pending, confirmed, shipped, delivered |
| region | TEXT | State abbreviation (NSW, VIC, QLD, etc.) |
| created_at | TEXT | ISO 8601 timestamp |

**Joins**: orders.customer_id → customers.id
**Note**: Historical orders before 2024 may have NULL region values.
```

### config/tools.ts — Domain Tools

Three exports required:

| Export | Type | Purpose |
|--------|------|---------|
| `DOMAIN_TOOL_DEFINITIONS` | `ToolDefinition[]` | Tool definitions for the AI |
| `executeDomainTool` | `async function` | Server-side tool executor |
| `domainToolInputSummary` | `function` | Human-readable input for streaming UI |

**When to add domain tools:**
- Calculations that are too complex for SQL alone
- Aggregations that span multiple queries
- Domain-specific metrics (e.g., Cpk, conversion rate, retention)
- Lookups from external sources

**When NOT to add domain tools:**
- Simple SELECT queries — the built-in `run_sql` handles these
- Basic calculations — the built-in `calculate` tool handles these
- Chart rendering — the built-in `create_chart` tool handles this

### config/nav.ts — Sidebar Navigation

Each section has a label and items array. Items need `to` (route path), `label`, and optional `icon` (Lucide icon component).

The CommandPalette (Ctrl+K) automatically reads `NAV_SECTIONS` — no separate configuration needed.

### config/seed.sql — KB Seed Data

Categories should match `APP_CONFIG.kbCategories`. Each article needs:
- `slug` — URL-friendly identifier (unique)
- `category` — one of the configured categories
- `title` — article title
- `content` — full markdown content

**Good KB articles to seed:**
- Business overview and key metrics
- Data dictionary (table/column explanations)
- FAQs about common data questions
- Methodology notes (how metrics are calculated)
- Getting started guide for end users

## Domain-Specific Code

### migrations/0002_domain.sql

Create domain tables. Follow D1/SQLite patterns:
- Use `TEXT` for dates (ISO 8601)
- Use `REAL` for decimals
- Use `INTEGER PRIMARY KEY AUTOINCREMENT` for IDs
- Add `created_at` and `updated_at` with `DEFAULT (datetime('now'))`
- **Never use** PostgreSQL functions (STDDEV, DATE_TRUNC, INTERVAL, etc.)

### src/client/pages/domain/Overview.tsx

Customise the landing dashboard. Available components:

| Component | Import | Purpose |
|-----------|--------|---------|
| `KPICard` | `@/components/dashboard/KPICard` | Metric card with trend |
| `FilterBar` | `@/components/dashboard/FilterBar` | Date range picker |
| `SkeletonCard` | `@/components/ui/Skeleton` | Loading placeholder |
| `SkeletonChart` | `@/components/ui/Skeleton` | Chart loading state |
| `SkeletonTable` | `@/components/ui/Skeleton` | Table loading state |

### src/server/routes/domain/example.ts

Add domain-specific API routes (dashboards, exports, etc.). These are mounted at `/api/domain/`.

## Deployment Checklist

- [ ] `ANTHROPIC_API_KEY` set via `wrangler secret put`
- [ ] D1 database created and ID in `wrangler.jsonc`
- [ ] Remote migrations applied (`npm run db:migrate:remote`)
- [ ] Remote seed data loaded (`npm run db:seed:remote`)
- [ ] Custom domain configured (optional)
- [ ] Lead capture toggled appropriately
- [ ] AI analyst tested with a real domain question

## Colour Schemes

Six built-in colour schemes (each with light/dark variants):

| Scheme | Primary colour | Best for |
|--------|---------------|----------|
| `default` (zinc) | Neutral grey | Professional, conservative |
| `blue` | Blue | Tech, corporate |
| `green` | Emerald | Health, environment, finance |
| `orange` | Orange | Energy, construction, trades |
| `violet` | Purple | Creative, premium |
| `rose` | Pink/rose | Beauty, wellness, lifestyle |

Users choose in Settings. The scheme is stored in localStorage and applied via CSS custom properties.
