# Permission Presets

Curated permission presets for `settings.local.json`. Each preset is a JSON array of permission strings grouped with `//` comments. Compose presets by stacking: Universal Base + language preset + deployment preset + extras.

## Syntax Reference

| Pattern | Meaning |
|---------|---------|
| `Bash(git *)` | Space before `*` = word boundary. Matches `git status` but not `gitk`. **Preferred syntax.** |
| `Bash(git*)` | No space = prefix match. Matches `git status` AND `gitk`. |
| `Bash(nvidia-smi)` | Exact match — no arguments. Use for bare commands. |
| `WebFetch` | Blanket web fetch (all domains) |
| `WebFetch(domain:example.com)` | Domain-scoped web fetch |
| `WebSearch` | Blanket web search |
| `mcp__*` | All MCP servers and tools |
| `mcp__brain__*` | All tools on one MCP server |
| `mcp__brain__brain_sites` | One specific MCP tool |

### Format Notes

- **Deprecated**: `Bash(git:*)` colon syntax still works but prefer space syntax `Bash(git *)`
- **"Don't ask again"** prompt uses legacy colon format (e.g. `node:*`) — it's equivalent but looks different
- **Comments**: `"// --- Section ---"` strings in the allow array are ignored and useful for organisation
- **Not hot-reloaded**: Changes to `settings.local.json` require a session restart. "Don't ask again" bypasses this because it injects into the running session directly.

**Critical**: Project `settings.local.json` **SHADOWS** global settings (does not merge). If a project has its own allow list, the global allow list is ignored entirely for that project.

Shell operators (`&&`, `||`, `;`) are handled safely — `Bash(git *)` won't match `git add && rm -rf /`.

---

## Universal Base

Every project gets these. Version control, file operations, and basic tools needed for all development.

```json
"// --- Version Control ---",
"Bash(git *)",
"Bash(gh *)",

"// --- File Operations ---",
"Bash(ls *)",
"Bash(cat *)",
"Bash(head *)",
"Bash(tail *)",
"Bash(wc *)",
"Bash(sort *)",
"Bash(mkdir *)",
"Bash(cp *)",
"Bash(mv *)",
"Bash(touch *)",
"Bash(chmod *)",
"Bash(find *)",
"Bash(tree *)",
"Bash(du *)",
"Bash(df *)",

"// --- Archives ---",
"Bash(tar *)",
"Bash(zip *)",
"Bash(unzip *)",
"Bash(gzip *)",

"// --- Text Processing ---",
"Bash(grep *)",
"Bash(rg *)",
"Bash(awk *)",
"Bash(sed *)",
"Bash(diff *)",
"Bash(jq *)",
"Bash(echo *)",
"Bash(printf *)",
"Bash(tee *)",

"// --- System ---",
"Bash(which *)",
"Bash(whereis *)",
"Bash(type *)",
"Bash(ps *)",
"Bash(env *)",
"Bash(date *)",
"Bash(uname *)",
"Bash(make *)",

"// --- Network ---",
"Bash(curl *)",
"Bash(wget *)",
"Bash(ssh *)",
"Bash(scp *)",
"Bash(rsync *)",
"Bash(dig *)",

"// --- Web ---",
"WebSearch",
"WebFetch"
```

---

## JavaScript / TypeScript

For any JS/TS project. Add to Universal Base.

```json
"// --- Node.js ---",
"Bash(node *)",
"Bash(npm *)",
"Bash(npx *)",

"// --- Alternative Runtimes ---",
"Bash(bun *)",
"Bash(bunx *)",
"Bash(deno *)",

"// --- Package Managers ---",
"Bash(pnpm *)",
"Bash(yarn *)",

"// --- TypeScript ---",
"Bash(tsc *)",
"Bash(tsx *)",

"// --- Bundlers ---",
"Bash(esbuild *)",
"Bash(vite *)",
"Bash(turbo *)",

"// --- Testing ---",
"Bash(jest *)",
"Bash(vitest *)",
"Bash(playwright *)",
"Bash(cypress *)",

"// --- Linting / Formatting ---",
"Bash(eslint *)",
"Bash(prettier *)",
"Bash(biome *)"
```

---

## Python

For Python projects. Add to Universal Base.

```json
"// --- Python Runtime ---",
"Bash(python *)",
"Bash(python3 *)",

"// --- Package Managers ---",
"Bash(pip *)",
"Bash(pip3 *)",
"Bash(uv *)",
"Bash(poetry *)",
"Bash(pipx *)",
"Bash(conda *)",

"// --- Testing / Quality ---",
"Bash(pytest *)",
"Bash(mypy *)",
"Bash(ruff *)",
"Bash(black *)",
"Bash(flake8 *)",
"Bash(isort *)",

"// --- Dev Servers ---",
"Bash(flask *)",
"Bash(uvicorn *)",
"Bash(gunicorn *)",
"Bash(django-admin *)",

"// --- Notebooks ---",
"Bash(jupyter *)"
```

---

## PHP

For PHP projects including WordPress and Laravel. Add to Universal Base.

```json
"// --- PHP Runtime ---",
"Bash(php *)",
"Bash(composer *)",

"// --- WordPress ---",
"Bash(wp *)",

"// --- Testing / Quality ---",
"Bash(phpunit *)",
"Bash(phpstan *)",
"Bash(phpcs *)",
"Bash(phpcbf *)",
"Bash(pest *)",

"// --- Laravel ---",
"Bash(artisan *)",
"Bash(sail *)"
```

---

## Go

For Go projects. Add to Universal Base.

```json
"// --- Go ---",
"Bash(go *)",
"Bash(golangci-lint *)"
```

---

## Rust

For Rust projects. Add to Universal Base.

```json
"// --- Rust ---",
"Bash(cargo *)",
"Bash(rustc *)",
"Bash(rustup *)"
```

---

## Ruby

For Ruby / Rails projects. Add to Universal Base.

```json
"// --- Ruby ---",
"Bash(ruby *)",
"Bash(gem *)",
"Bash(bundle *)",
"Bash(bundler *)",
"Bash(rails *)",
"Bash(rake *)",
"Bash(rspec *)"
```

---

## Cloudflare Worker

Deployment preset. Add to Universal Base + JavaScript/TypeScript.

```json
"// --- Wrangler ---",
"Bash(wrangler *)",
"Bash(npx wrangler *)"
```

---

## Vercel

Deployment preset. Add to Universal Base + JavaScript/TypeScript.

```json
"// --- Vercel ---",
"Bash(vercel *)",
"Bash(npx vercel *)",

"// --- Prisma (common with Vercel) ---",
"Bash(prisma *)",
"Bash(npx prisma *)"
```

---

## Docker / Containers

For containerised projects. Add to any stack.

```json
"// --- Docker ---",
"Bash(docker *)",
"Bash(docker-compose *)",

"// --- Kubernetes ---",
"Bash(kubectl *)",
"Bash(helm *)",

"// --- IaC ---",
"Bash(terraform *)",
"Bash(pulumi *)"
```

---

## Database

For projects that interact with databases directly. Add to any stack.

```json
"// --- SQL ---",
"Bash(psql *)",
"Bash(mysql *)",
"Bash(sqlite3 *)",

"// --- NoSQL ---",
"Bash(redis-cli *)",
"Bash(mongosh *)"
```

---

## Cloud CLIs

For cloud-deployed projects. Add to any stack.

```json
"// --- AWS ---",
"Bash(aws *)",

"// --- Google Cloud ---",
"Bash(gcloud *)",
"Bash(gsutil *)",

"// --- Azure ---",
"Bash(az *)"
```

---

## AI / GPU

For AI/ML workloads. Add to any stack.

```json
"// --- Local LLM ---",
"Bash(ollama *)",

"// --- GPU ---",
"Bash(nvidia-smi *)",
"Bash(nvidia-smi)"
```

Note: `Bash(nvidia-smi)` (no wildcard) matches the bare command with no arguments, which is the most common usage.

---

## MCP Servers

MCP (Model Context Protocol) servers provide tool access to external services. Permission patterns use the format `mcp__servername__toolname`.

### Blanket Allow (Recommended)

If you trust all your connected MCP servers, use a single wildcard:

```json
"// --- All MCP Servers ---",
"mcp__*"
```

This covers every MCP server and tool. Since you control which servers are connected (via `.mcp.json` or Claude Desktop config), this is safe for most setups.

### Per-Server Wildcards

For granular control, allow all tools on specific servers:

```json
"mcp__brain__*",
"mcp__playwright__*",
"mcp__vault__*"
```

### Individual Tools

For maximum control, allow specific tools only:

```json
"mcp__brain__brain_sites",
"mcp__brain__brain_recall"
```

---

## Combining Presets

Presets stack. Examples:

| Project Type | Presets to Combine |
|-------------|-------------------|
| Next.js on Vercel | Universal + JavaScript/TypeScript + Vercel |
| Cloudflare Worker | Universal + JavaScript/TypeScript + Cloudflare Worker |
| Django app | Universal + Python + Database + Docker |
| WordPress plugin | Universal + PHP |
| Rust CLI | Universal + Rust |
| ML project | Universal + Python + AI/GPU |
| Full-stack ops | Universal + JavaScript/TypeScript + Python + Docker + Database + MCP (blanket) |

When merging, deduplicate and keep the grouped `//` comment structure. The final `settings.local.json` should look like:

```json
{
  "permissions": {
    "allow": [
      "// --- Version Control ---",
      "Bash(git *)",
      "Bash(gh *)",
      "// --- Node.js ---",
      "Bash(node *)",
      "..."
    ],
    "deny": []
  }
}
```
