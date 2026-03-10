# Flare AI App — Setup Pattern

Reference for the operations Claude performs when scaffolding a new AI app from
vite-flare-ai-starter. Adapt to the user's environment.

## Step 1: Clone and Clean

```bash
git clone https://github.com/jezweb/vite-flare-ai-starter.git PROJECT_DIR --depth 1
cd PROJECT_DIR
rm -rf .git
git init
```

## Step 2: Find-Replace Targets

Replace default values with the project-specific names:

| File | Target | Replace with | Notes |
|------|--------|-------------|-------|
| `wrangler.jsonc` | `"my-ai-app"` (name field) | `"PROJECT_NAME"` | Worker name |
| `wrangler.jsonc` | `"my-ai-app-db"` (database_name) | `"PROJECT_NAME-db"` | |
| `wrangler.jsonc` | `REPLACE_WITH_YOUR_DATABASE_ID` | Actual database_id | After creating D1 |
| `wrangler.jsonc` | Existing `account_id` value | Verify or remove | Let wrangler prompt |
| `package.json` | `"vite-flare-ai-starter"` | `"PROJECT_NAME"` | Package name |
| `package.json` | Version | `"0.1.0"` | Reset version |
| `index.html` | Favicon emoji and `<title>` content | Client's branding | |

## Step 3: Configure App Identity

Edit `config/app.ts`:

```typescript
export const APP_CONFIG = {
  name: "Client AI",           // Display name
  description: "AI analytics", // One-liner
  logoChar: "C",               // Sidebar logo character
  version: "1.0.0",
  defaultModel: "claude-sonnet" as const,
  timezone: "Australia/Sydney",
  allowedOrigins: [],
  kbCategories: ["general", "technical", "faq"],
  features: {
    leadCapture: false,        // Set true for demo mode
  },
};
```

## Step 4: Create .dev.vars

```
# Anthropic API Key (required for AI features)
ANTHROPIC_API_KEY=sk-ant-api03-your-key-here
```

## Step 5: Create Cloudflare D1 Database

```bash
npx wrangler d1 create PROJECT_NAME-db
# Extract database_id from output, update wrangler.jsonc
```

## Step 6: Install, Migrate, Seed

```bash
npm install
npm run db:migrate
npm run db:seed
```

## Step 7: Configure Domain

This is the critical customisation step. Edit these files:

### config/schema.ts — AI System Prompt

The `DOMAIN_SYSTEM_PROMPT` string is injected into the AI's system prompt. Include:
- Business description and terminology
- Database table schemas with column descriptions
- Join paths between tables
- Common query patterns
- Data quality caveats

### config/tools.ts — Domain AI Tools

Export `DOMAIN_TOOL_DEFINITIONS` (array of tool definitions for the AI), `executeDomainTool` (server-side executor), and `domainToolInputSummary` (human-readable input display).

### config/nav.ts — Sidebar Navigation

Define `NAV_SECTIONS` with items pointing to routes. The CommandPalette reads these automatically.

### config/seed.sql — KB Articles

Insert domain-specific knowledge base articles. Categories should match `APP_CONFIG.kbCategories`.

## Step 8: Domain Migration (if needed)

Create `migrations/0002_domain.sql` with domain-specific tables, then:

```bash
npm run db:migrate
```

## Step 9: Initial Commit

```bash
git add -A
git commit -m "Initial commit: PROJECT_NAME from vite-flare-ai-starter"
```

## Step 10: Push to GitHub

```bash
gh repo create jezweb/PROJECT_NAME --private --source=. --push --description "AI-powered analytics for CLIENT_NAME"
```

## macOS sed Note

macOS `sed -i` requires an extension argument: `sed -i '' 's/old/new/g' file`.
Use the Edit tool directly (preferred — avoids sed entirely).
