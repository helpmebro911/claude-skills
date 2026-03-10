---
name: flare-ai-app
description: "Scaffold an AI-powered business intelligence app from vite-flare-ai-starter. Includes AI tool-calling agent with SSE streaming, knowledge base with FTS5, inline charts, user memories, conversation sharing, and lead capture. Clone, configure domain tools + schema, seed KB, deploy to Cloudflare."
compatibility: claude-code-only
---

# Flare AI App

Clone and configure the AI business intelligence starter into a standalone client app. Produces a fully branded, deployable AI analyst platform with domain-specific tools, knowledge base, and dashboards.

## What You Get

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, Vite, Tailwind v4, shadcn/ui colour schemes |
| Backend | Hono (on Cloudflare Workers) |
| Database | D1 (SQLite at edge) with FTS5 knowledge base search |
| AI | Anthropic Claude API (direct, with prompt caching + SSE streaming) |
| Charts | Recharts (bar, line, pie, scatter, control) inline in chat |
| Features | AI analyst, KB, memories, sharing, artifacts, lead gate, Ctrl+K search |
| Deployment | Cloudflare Workers with Static Assets |

See [references/tech-stack.md](references/tech-stack.md) for the full dependency list.

## Architecture

Everything in `src/` is shared infrastructure — never needs modification. Everything in `config/` is domain-specific — this is what you customise per client.

```
config/           ← CUSTOMISE per client
  app.ts          — app identity, feature toggles, defaults
  schema.ts       — AI system prompt (domain knowledge, table schemas)
  tools.ts        — domain-specific AI tools (definitions + executors)
  nav.ts          — sidebar navigation sections
  seed.sql        — initial KB articles

src/server/       ← STABLE: shared AI agent infrastructure
src/client/       ← STABLE: shared UI components
```

## Workflow

### Step 1: Gather Project Info

Ask for:

| Required | Optional |
|----------|----------|
| Project name (kebab-case, e.g. `acme-ai`) | Custom domain |
| Client/business name (display name) | Logo character/emoji |
| Business domain (what data? what industry?) | Colour scheme preference |
| One-line description | Lead capture (demo mode)? |

Also ask about the data source:
- **D1 only** (default): Domain tables in D1, populated via import/seed
- **Hyperdrive**: Existing PostgreSQL/MySQL database to connect to

### Step 2: Clone and Configure

See [references/setup-pattern.md](references/setup-pattern.md) for exact operations.

1. **Clone** the starter repo and remove `.git`, init fresh repo
   ```bash
   git clone https://github.com/jezweb/vite-flare-ai-starter.git PROJECT_DIR --depth 1
   cd PROJECT_DIR
   rm -rf .git
   git init
   ```

2. **Rebrand** — find-replace in these files:

   | File | What Changes |
   |------|-------------|
   | `wrangler.jsonc` | Worker name (`"name"`), database name |
   | `package.json` | Package name, version to `0.1.0` |
   | `index.html` | `<title>`, favicon emoji |
   | `config/app.ts` | `name`, `description`, `logoChar`, `version`, `defaultModel`, `kbCategories` |

3. **Create** `.dev.vars` from template with `ANTHROPIC_API_KEY`

4. **Create** Cloudflare D1 database and update `wrangler.jsonc` with `database_id`

5. **Install** and migrate:
   ```bash
   npm install
   npm run db:migrate
   npm run db:seed
   ```

### Step 3: Configure Domain (THE KEY STEP)

This is where the app becomes specific to the client's business. Configure these 5 files:

#### 3a. Domain Schema (`config/schema.ts`)

Write the `DOMAIN_SYSTEM_PROMPT` that makes the AI smart about THIS client's data:

```typescript
export const DOMAIN_SYSTEM_PROMPT = `
## Domain: [Client's Business]

You are an AI analyst for [Client Name], a [business description].

### Database Tables

**orders** — Customer orders
| Column | Type | Description |
|--------|------|-------------|
| id | INTEGER | Primary key |
| customer_name | TEXT | Customer name |
| total | REAL | Order total in AUD |
| status | TEXT | pending/confirmed/shipped/delivered |
| created_at | TEXT | ISO timestamp |

### Domain Knowledge
- [Key business rules, terminology, seasonal patterns]
- [How metrics are calculated]
- [Important relationships between tables]

### Common Queries
- "How are sales this month?" → Query orders table, group by date
- "Who are our top customers?" → Aggregate by customer_name
`;
```

**Tips**: Include table schemas with column descriptions, domain terminology, join paths between tables, and example query patterns. This is the single most important customisation.

#### 3b. Domain Tools (`config/tools.ts`)

Define AI tools beyond the built-in ones (run_sql, search_kb, create_chart, etc.):

```typescript
import type { ToolDefinition } from "@shared/types";

export const DOMAIN_TOOL_DEFINITIONS: ToolDefinition[] = [
  {
    name: "get_summary_stats",
    description: "Get summary statistics for a date range",
    input_schema: {
      type: "object" as const,
      properties: {
        from_date: { type: "string", description: "Start date (YYYY-MM-DD)" },
        to_date: { type: "string", description: "End date (YYYY-MM-DD)" },
      },
      required: ["from_date", "to_date"],
    },
  },
];

export async function executeDomainTool(
  name: string,
  input: Record<string, unknown>,
  db: D1Database,
): Promise<unknown> {
  switch (name) {
    case "get_summary_stats": {
      // Execute domain-specific queries
      const result = await db.prepare(
        "SELECT COUNT(*) as count, SUM(total) as revenue FROM orders WHERE created_at >= ?1 AND created_at <= ?2"
      ).bind(input.from_date, input.to_date).first();
      return result;
    }
    default:
      return { error: `Unknown tool: ${name}` };
  }
}

export function domainToolInputSummary(
  name: string,
  input: Record<string, unknown>,
): string | null {
  if (name === "get_summary_stats") return `${input.from_date} to ${input.to_date}`;
  return null;
}
```

#### 3c. Navigation (`config/nav.ts`)

Add sidebar sections for domain-specific pages:

```typescript
import { LayoutDashboard, Sparkles, BookOpen, Settings } from "lucide-react";
import type { LucideIcon } from "lucide-react";

export interface NavItem { to: string; label: string; icon?: LucideIcon; }
export interface NavSection { label: string; items: NavItem[]; }

export const NAV_SECTIONS: NavSection[] = [
  {
    label: "Analytics",
    items: [
      { to: "/app", label: "Overview", icon: LayoutDashboard },
      { to: "/app/analyst", label: "AI Analyst", icon: Sparkles },
    ],
  },
  {
    label: "Resources",
    items: [
      { to: "/app/kb", label: "Knowledge Base", icon: BookOpen },
      { to: "/app/settings", label: "Settings", icon: Settings },
    ],
  },
];
```

#### 3d. Seed Data (`config/seed.sql`)

Insert KB articles with domain-specific knowledge the AI can search:

```sql
INSERT INTO kb_articles (slug, category, title, content) VALUES
('business-overview', 'general', 'Business Overview',
 'Description of the business, its products/services, key metrics...'),
('data-dictionary', 'technical', 'Data Dictionary',
 'Explanation of database tables, column meanings, data quality notes...');
```

#### 3e. App Identity (`config/app.ts`)

Update the APP_CONFIG with client branding:

```typescript
export const APP_CONFIG = {
  name: "Acme AI",
  description: "AI-powered analytics for Acme Corp",
  logoChar: "A",
  version: "1.0.0",
  defaultModel: "claude-sonnet",
  // ...
};
```

### Step 4: Create Domain Migration (if needed)

If the client has domain-specific tables beyond the built-in ones:

```bash
# Create migration file
cat > migrations/0002_domain.sql << 'EOF'
-- Domain tables for [Client]
CREATE TABLE IF NOT EXISTS orders (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  customer_name TEXT NOT NULL,
  total REAL NOT NULL DEFAULT 0,
  status TEXT NOT NULL DEFAULT 'pending',
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);
EOF

# Apply
npm run db:migrate
```

### Step 5: Create Domain Dashboard (optional)

Edit `src/client/pages/domain/Overview.tsx` to show KPI cards and charts specific to the client's business. Use the included components:

```tsx
import { KPICard } from "@/components/dashboard/KPICard";
import { FilterBar } from "@/components/dashboard/FilterBar";
import { SkeletonCard } from "@/components/ui/Skeleton";
```

### Step 6: Verify Locally

```bash
npm run dev
```

Check:
- [ ] App loads with correct branding at http://localhost:5173 (or next available port)
- [ ] AI analyst responds (requires ANTHROPIC_API_KEY in .dev.vars)
- [ ] Knowledge base shows seeded articles
- [ ] Ctrl+K command palette works
- [ ] Theme switching works in Settings

### Step 7: Deploy to Production

```bash
# Set secrets
echo "sk-ant-api03-your-key" | npx wrangler secret put ANTHROPIC_API_KEY

# Migrate remote database
npm run db:migrate:remote
npm run db:seed:remote

# Build and deploy
npm run deploy
```

After first deploy:
- [ ] Verify at `https://PROJECT_NAME.workers.dev`
- [ ] Test AI analyst with a real question
- [ ] Optionally add custom domain via `wrangler.jsonc` routes

## Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| AI returns errors | Missing ANTHROPIC_API_KEY | Add key to `.dev.vars` (local) or `wrangler secret put` (production) |
| KB search returns nothing | Seed data not loaded | Run `npm run db:seed` |
| D1 "table not found" | Migrations not applied | Run `npm run db:migrate` (local) or `npm run db:migrate:remote` (production) |
| 500 on API endpoints | Dev server started before migrations | Restart dev server after applying migrations |
| Charts not rendering | Missing data in tool response | Check `create_chart` tool returns valid `data`, `x_key`, `y_keys` |
| Model selector shows "unavailable" | No API key configured | Expected without key — set ANTHROPIC_API_KEY |
| Lead gate blocks access | `features.leadCapture` is true | Set to `false` in config/app.ts, or register via the gate |
| Build > 500KB warning | Recharts is large (~600KB) | Expected — use React.lazy() for code splitting if needed |

## What Gets Customised

| File | What Changes | When |
|------|-------------|------|
| `config/app.ts` | App name, description, logo, features | Always |
| `config/schema.ts` | AI system prompt with domain knowledge | Always |
| `config/tools.ts` | Domain-specific AI tool definitions | Usually |
| `config/nav.ts` | Sidebar navigation items | Usually |
| `config/seed.sql` | KB articles for the domain | Always |
| `wrangler.jsonc` | Worker name, database_id, custom domain | Always |
| `package.json` | Package name | Always |
| `index.html` | Title, favicon | Always |
| `migrations/0002_domain.sql` | Domain-specific tables | If D1 data |
| `src/client/pages/domain/Overview.tsx` | Domain dashboard | Optional |

## Built-in AI Tools (always available)

| Tool | Purpose |
|------|---------|
| `list_tables` | Show available tables + row counts |
| `get_table_schema` | Column names, types, sample data |
| `run_sql` | Read-only SQL (up to 500 rows) |
| `calculate` | Arithmetic expressions |
| `search_kb` | FTS5 knowledge base search |
| `create_chart` | Inline chart (bar/line/pie/scatter/control) |
| `calculate_process_stats` | Mean, stddev, Cp, Cpk |

## Domain Tool Libraries (copy from previous projects)

For common verticals, reuse tool definitions from existing projects:

| Vertical | Source | Tools included |
|----------|--------|---------------|
| Manufacturing | Crosbe AI / Plastiq AI | SPC, Nelson Rules, Cpk, spec limits, batch tracking |
| SEO | WhisperingWombat | Rank tracking, crawl analysis, competitor comparison |
| General analytics | This starter | Summary stats, date range queries |

See [references/customisation-guide.md](references/customisation-guide.md) for the full guide.
