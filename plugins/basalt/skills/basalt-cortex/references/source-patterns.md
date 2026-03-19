# Source Extraction Patterns

Per-source patterns for fetching, pre-filtering, and extracting knowledge into Basalt format. Claude generates a script from these patterns adapted to the user's environment.

---

## Gmail

**Prerequisites**: Gmail MCP connected (`google-gmail-jez` or `google-gmail`)

**Fetch pattern (MCP — preferred)**:

```
# Phase 1a: Extract contacts (quick scan, 100 emails)
gmail_messages({ action: "extract_contacts", query: "in:inbox after:YYYY/MM/DD", field: "from",
  excludeDomains: ["jezweb.net", "google.com", "github.com", "cloudflare.com", ...], maxResults: 100 })

# Phase 1a (sent): Who Jez emails
gmail_messages({ action: "extract_contacts", query: "in:sent after:YYYY/MM/DD", field: "to",
  excludeDomains: [...], maxResults: 100 })

# Phase 1b: List emails in monthly batches
gmail_messages({ action: "list",
  query: "in:inbox -category:promotions -category:social -category:updates -category:forums after:YYYY/MM/DD before:YYYY/MM/DD",
  maxResults: 50, format: "compact", bodyPreview: 1000 })

# Phase 1c: Get full content for significant threads
gmail_messages({ action: "get", messageId: "...", bodyFormat: "text" })
```

**Pre-filter**: See prefilter-patterns.md. From the `list` results, skip:
- Synergy Wholesale 2FA, eligibility reports, transaction statements
- CrazyTel auto top-ups
- Domain expiry/transfer/registration notifications
- Wordfence login alerts
- L2Chat/Bot notification emails
- Payment receipts from automated systems

**Extract**: Claude does AI extraction in-context while scanning list results. No external LLM call needed. Identify clients, contacts, communications, and knowledge facts from snippets + full bodies.

**Basalt mapping**:
- Each unique business domain → `clients/{domain}.md`
- Each unique person → `contacts/{first-last}.md`
- Each significant thread → `communications/{YYYY}/{MM}/{YYYY-MM-DD}-{subject-slug}.md`
- Each extracted fact → `knowledge/{topic-slug}.md`

**Idempotency**: Use Gmail thread ID as `source_id`. Check `state.json` processed list before processing.

**Cursor**: Store the newest thread timestamp in `state.json` cursors.gmail.

**Batch strategy**: Process one month at a time. For large mines (full year), fetch 50 emails per month-batch, process, then move to the next month. Generate a single Python script with all extracted entities at the end.

---

## Google Chat

**Prerequisites**: Google Chat MCP connected (`google-chat`)

**Fetch pattern (MCP)**:

```
# List all spaces
chat_spaces({ action: "list", filter: "all", limit: 50 })

# Categorise spaces — skip bot-only spaces:
# SKIP: Email Copilot, Coffee, Chat Copilot, Flux Copilot, Paste Bin
# MINE: Client spaces, business ops, team discussions

# Fetch messages ONE SPACE AT A TIME
chat_messages({ action: "list", spaceId: "ABC123", range: "month", limit: 50 })

# NEVER use search_active for mining — it fires all spaces in parallel
# and times out on accounts with 50+ spaces (Jezweb has 57)
```

**Pre-filter**: Skip bot messages (`senderType: "BOT"`), join/leave events, webhook posts, reactions-only messages, messages under 20 chars.

**Extract**: Group messages into conversation threads (by `threadId`). Extract decisions, action items, knowledge. Claude does extraction in-context.

**Basalt mapping**:
- Each conversation thread → `communications/{YYYY}/{MM}/{YYYY-MM-DD}-{subject-slug}.md` with `channel: google_chat`
- Decisions and facts → `knowledge/{topic-slug}.md`
- New participants → `contacts/{first-last}.md`

**Idempotency**: Use message ID as `source_id`.

**Space-by-space iteration**: Process spaces in order of business value:
1. Client spaces (Verge, ACPS, Procool, etc.)
2. Business ops (Help Desk, Website Sales, Accounts Receivable)
3. Team/internal (Team Jezweb, HR, Finances)
4. Dev/AI (Claude Code, MCP Servers, AI Agents)
Save progress after each space so failures are resumable.

---

## Slack

**Prerequisites**: Slack MCP connected OR Slack API token

**Fetch pattern (MCP)**:
```
Use Slack MCP tools to list channels, read messages
```

**Fetch pattern (API)**:
```bash
curl -s "https://slack.com/api/conversations.list" -H "Authorization: Bearer $SLACK_TOKEN"
curl -s "https://slack.com/api/conversations.history?channel=CHANNEL_ID&limit=50" -H "Authorization: Bearer $SLACK_TOKEN"
```

**Pre-filter**: Skip bot messages, app notifications, channel join/leave.

**Extract**: Same as Google Chat — group by thread, extract decisions and knowledge.

**Basalt mapping**: Same as Google Chat with `channel: slack`, `source: slack`.

---

## Google Drive

**Prerequisites**: `gws` CLI authenticated with drive scope

**Fetch pattern**:
```bash
# List recent files
gws drive files list --query "modifiedTime > '2026-01-01'" --format json

# Get file metadata
gws drive files get FILE_ID --format json

# Download text content (for Google Docs)
gws drive files export FILE_ID --mime-type text/plain
```

**Pre-filter**: Skip files smaller than 100 bytes, skip image/video/audio files.

**Extract**: For documents, extract the title, summary, owner, shared-with list. Don't mine the full content unless asked — Drive documents are better linked than copied.

**Basalt mapping**:
- Each document → `knowledge/know_{ts}_{hash}.md` with document URL in `source_url`
- Shared-with lists → update `contacts/` with collaboration relationships
- Folders → `projects/` if they represent a project

---

## Local Folders

**Prerequisites**: None

**Fetch pattern**:
```bash
# Find markdown files in a directory
find /path/to/folder -name "*.md" -type f

# Read each file
cat /path/to/file.md
```

**Pre-filter**: Skip files in `node_modules/`, `.git/`, `dist/`, `.wrangler/`.

**Extract**: Read frontmatter if present. If no frontmatter, extract title from first heading. Generate a summary.

**Basalt mapping**:
- Each file → type depends on content (note, knowledge, project)
- Preserve existing frontmatter where compatible with Basalt format
- Add `source: local`, `source_url: file:///path/to/original`

---

## MCP Servers

**Prerequisites**: MCP server connected (Brain, Vault, or any custom server)

**Fetch pattern**:
```
# Brain MCP
brain_recall query="all knowledge" limit=100
brain_knowledge list limit=100

# Vault MCP
vault recall query="all items"
vault knowledge_list
```

**Pre-filter**: Skip items already in `~/.cortex/` by checking `source_id`.

**Extract**: MCP items often already have structured fields. Map directly to Basalt frontmatter.

**Basalt mapping**:
- Brain knowledge → `knowledge/know_{ts}_{hash}.md` with `source: brain`
- Brain clients → `clients/{domain}.md` with `source: brain`
- Vault items → `knowledge/know_{ts}_{hash}.md` with `source: vault`

---

## Web Pages

**Prerequisites**: WebFetch tool, Firecrawl MCP, or browser automation

**Fetch pattern**:
```
# WebFetch
WebFetch url="https://example.com/article"

# Firecrawl
firecrawl_scrape url="https://example.com" format=markdown

# Browser
playwright-cli -s=cortex open https://example.com
playwright-cli -s=cortex snapshot
```

**Pre-filter**: Skip pages with less than 200 chars of content (likely JS-rendered blanks).

**Extract**: Extract title, author, date, summary. For reference material, keep the full content.

**Basalt mapping**:
- Each page → `knowledge/know_{ts}_{hash}.md` or `notes/{slug}.md`
- `source: web`, `source_url: https://...`

---

## Calendar

**Prerequisites**: `gws` CLI authenticated with calendar scope

**Fetch pattern**:
```bash
gws calendar events list --time-min 2026-01-01 --max-results 100 --format json
```

**Pre-filter**: Skip all-day events without descriptions, skip declined events.

**Extract**: Extract meeting title, attendees, description/agenda, recurring pattern.

**Basalt mapping**:
- Each meeting → `communications/{YYYY}/{MM}/comm_{ts}_{hash}.md` with `channel: calendar`
- New attendees → `contacts/{slug}.md`
- Meeting notes in description → `knowledge/know_{ts}_{hash}.md`

---

## Obsidian Vault Import

**Prerequisites**: Path to existing Obsidian vault

**Fetch pattern**:
```bash
find /path/to/vault -name "*.md" -not -path "*/.obsidian/*" -not -path "*/.trash/*"
```

**Pre-filter**: Skip Obsidian system files (`.obsidian/`, `.trash/`).

**Extract**: Read existing frontmatter. Normalise to Basalt format:
- Add `id` if missing (generate from filename)
- Add `type` if missing (infer from folder name or content)
- Add `source: obsidian`, `source_url: obsidian://open?vault=NAME&file=PATH`
- Preserve existing `tags` and `aliases` (Obsidian-native fields)
- Generate `summary` if missing

**Basalt mapping**: Preserve the file as-is with normalised frontmatter. Copy to `~/.cortex/notes/` unless the folder maps to a known type (clients/, contacts/, etc.).
