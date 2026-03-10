# Flare AI App — Tech Stack

## Runtime & Server

| Package | Purpose |
|---------|---------|
| `hono` | HTTP framework (Cloudflare Workers) |
| `@cloudflare/vite-plugin` | Dev server + build integration |
| `wrangler` | Cloudflare CLI (deploy, D1, secrets) |

## Frontend

| Package | Purpose |
|---------|---------|
| `react` + `react-dom` | UI framework (v19) |
| `react-router-dom` | Client-side routing (v7) |
| `@tanstack/react-query` | Server state management |
| `tailwindcss` + `@tailwindcss/vite` | Utility CSS (v4) |
| `class-variance-authority` | Component variant utility |
| `clsx` + `tailwind-merge` | Class name merging |
| `lucide-react` | Icons |

## AI & Data Visualisation

| Package | Purpose |
|---------|---------|
| Anthropic Claude API | AI agent (direct HTTP, not SDK) |
| `recharts` | Charts (bar, line, pie, scatter, control) |
| `react-markdown` + `remark-gfm` | Markdown rendering in chat |

## Date Handling

| Package | Purpose |
|---------|---------|
| `date-fns` | Date formatting utilities |

## Build & Dev

| Package | Purpose |
|---------|---------|
| `vite` | Build tool (v6) |
| `@vitejs/plugin-react` | React Fast Refresh |
| `typescript` | Type checking |
| `@cloudflare/workers-types` | Worker type definitions |

## Cloudflare Services Used

| Service | Binding | Purpose |
|---------|---------|---------|
| D1 | `DB` | SQLite database (structured data + FTS5 KB) |
| Workers AI | `AI` | Office doc preprocessing (toMarkdown) |
| Static Assets | `ASSETS` | SPA hosting |

## Optional (uncomment in wrangler.jsonc)

| Service | Binding | Purpose |
|---------|---------|---------|
| Hyperdrive | `HYPERDRIVE` | External PostgreSQL/MySQL connection pooling |
| R2 | — | File/object storage |
