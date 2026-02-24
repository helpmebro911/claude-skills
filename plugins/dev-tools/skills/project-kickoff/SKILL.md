---
name: project-kickoff
description: "Bootstrap new projects with curated settings.local.json permissions, CLAUDE.md, and .gitignore. Detects project type (cloudflare-worker, vercel-app, node-generic, python, ops-admin) and generates grouped, commented permission presets. Also tidies existing messy settings files (removes leaked secrets, shell fragments, deprecated MCP refs, duplicate entries). Trigger with 'kickoff', 'new project', 'bootstrap', 'setup claude', 'tidy permissions', 'clean settings', or 'init project'."
compatibility: claude-code-only
---

# Project Kickoff

Bootstrap a new project or clean up an existing one so Claude Code has the right permissions, documentation, and git setup from the start.

**Problem**: Every new project accumulates dozens of permission approvals one click at a time, resulting in bloated settings files with leaked secrets, shell fragments, and deprecated MCP refs. This skill generates HQ-quality project scaffolding upfront.

**Output**: `settings.local.json`, `CLAUDE.md`, `.gitignore` (and optionally git init + GitHub repo).

## Operating Modes

### Mode 1: New Project Setup

**When**: Starting a new project, or working in a directory without `.claude/settings.local.json`.

**Steps**:

1. **Detect project type** from files present in the directory:

   | Indicator | Type |
   |-----------|------|
   | `wrangler.jsonc` or `wrangler.toml` | cloudflare-worker |
   | `vercel.json` or `next.config.*` | vercel-app |
   | `package.json` (no deploy target) | javascript-typescript |
   | `pyproject.toml` or `setup.py` or `requirements.txt` | python |
   | `Cargo.toml` | rust |
   | `go.mod` | go |
   | `Gemfile` or `Rakefile` | ruby |
   | `composer.json` or `wp-config.php` | php |
   | `Dockerfile` or `docker-compose.yml` | docker |
   | `.claude/agents/` or operational scripts | ops-admin |
   | Empty directory | Ask the user |

   If ambiguous, ask. Types can stack (e.g. cloudflare-worker + javascript-typescript).

   Always include **Universal Base** preset. Add language and deployment presets based on detected type. Ask if the user wants MCP blanket (`mcp__*`) or per-server control.

2. **Generate `.claude/settings.local.json`**:
   - Read [references/permission-presets.md](references/permission-presets.md) for the preset definitions
   - Combine Universal Base + type-specific preset
   - Write with `//` comment groups for organisation
   - Warn the user: "Project settings.local.json SHADOWS your global settings — it does not merge"

3. **Generate `CLAUDE.md`**:
   - Read [references/claude-md-templates.md](references/claude-md-templates.md) for templates
   - Fill in: project name (from directory name or ask), today's date, detected stack
   - Pre-fill Jez's defaults (Cloudflare account ID, pnpm, EN-AU)

4. **Generate `.gitignore`**:
   - Use the type-appropriate template from [references/claude-md-templates.md](references/claude-md-templates.md)
   - Always include `.claude/settings.local.json` and `.dev.vars`

5. **Optionally** (ask first):
   - `git init` + first commit
   - `gh repo create jezweb/[name] --private` + push

### Mode 2: Tidy Existing Permissions

**When**: User says "tidy permissions", "clean settings", or the existing `settings.local.json` has more than ~50 entries.

**Steps**:

1. Run the tidy script to analyse the current file:
   ```bash
   python3 ${SKILL_DIR}/scripts/tidy_permissions.py .claude/settings.local.json
   ```

2. Review the report. It flags:
   - **Leaked secrets**: API keys, tokens, hex strings embedded in approval patterns
   - **Shell fragments**: `Bash(do)`, `Bash(fi)`, `Bash(then)`, `Bash(else)`, `Bash(done)`
   - **Deprecated MCP refs**: `mcp__bitwarden__*` and similar
   - **Duplicates**: Entries covered by a broader pattern already present
   - **One-time entries**: Entire commit messages, hardcoded paths that will never match again
   - **Consolidation opportunities**: e.g. 5 separate `Bash(git add:*)`, `Bash(git commit:*)` could become `Bash(git *)`

3. Present the cleaned version with a diff showing what changed.

4. Apply after user confirmation. Recommend the user rotate any leaked secrets.

### Mode 3: Add Preset

**When**: User says "add wrangler permissions" or "add MCP permissions" to an existing project.

**Steps**:

1. Read the relevant preset section from [references/permission-presets.md](references/permission-presets.md)
2. Read the existing `.claude/settings.local.json`
3. Merge without duplicating — add new entries, keep existing groups
4. Write the updated file

## Permission Syntax Quick Reference

| Pattern | Meaning |
|---------|---------|
| `Bash(git *)` | Preferred — space before `*` = word boundary |
| `Bash(nvidia-smi)` | Exact match, no arguments |
| `WebFetch` | Blanket web fetch (all domains) |
| `WebFetch(domain:x.com)` | Domain-scoped web fetch |
| `WebSearch` | Blanket web search |
| `mcp__*` | All MCP servers and tools |
| `mcp__brain__*` | All tools on one server |
| `mcp__brain__brain_sites` | One specific MCP tool |

**Deprecated**: `Bash(git:*)` colon syntax still works but prefer space syntax.

**Not hot-reloaded**: Edits to `settings.local.json` require a session restart. The "don't ask again" prompt injects at runtime (no restart needed) using legacy colon format — this is expected and both formats are equivalent.

**Critical**: Project `settings.local.json` **shadows** global settings. It does not merge. If a project has its own allow list, the global allow list is ignored entirely for that project.

Shell operators (`&&`, `||`, `;`) are handled safely — `Bash(git *)` won't match `git add && rm -rf /`.

## Autonomy

- **Just do it**: Detect project type, read existing files
- **Brief confirmation**: Write settings.local.json, CLAUDE.md, .gitignore (show what will be written)
- **Ask first**: git init, GitHub repo creation, overwriting existing files, applying tidy fixes

## Reference Files

| When | Read |
|------|------|
| Building permission presets | [references/permission-presets.md](references/permission-presets.md) |
| Generating CLAUDE.md and .gitignore | [references/claude-md-templates.md](references/claude-md-templates.md) |
