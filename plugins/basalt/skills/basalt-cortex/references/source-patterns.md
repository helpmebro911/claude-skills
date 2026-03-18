# Source Extraction Patterns

Per-source patterns for fetching, pre-filtering, and extracting knowledge into Basalt format. Claude generates a script from these patterns adapted to the user's environment.

---

## Gmail

**Prerequisites**: `gws` CLI authenticated (`gws auth login -s gmail`)

**Fetch pattern**:
```bash
# List threads (newest first, batch of 50)
gws gmail messages list --query "newer_than:30d" --max-results 50 --format json

# Get full thread content
gws gmail messages get MESSAGE_ID --format json
```

**Pre-filter**: See prefilter-patterns.md. Skip automated senders, marketing emails, notifications.

**Extract**: Send thread body to LLM. Extract contacts, client info, communication summary, knowledge facts.

**Basalt mapping**:
- Each unique sender domain → `clients/{domain}.md`
- Each unique email address → `contacts/{slug}.md`
- Each thread → `communications/{YYYY}/{MM}/comm_{ts}_{hash}.md`
- Each extracted fact → `knowledge/know_{ts}_{hash}.md`

**Idempotency**: Use Gmail thread ID as `source_id`. Check `state.json` processed list before processing.

**Cursor**: Store the newest thread timestamp in `state.json` cursors.gmail. Next run starts from there.

---

## Google Chat

**Prerequisites**: `gws` CLI authenticated with chat scope (`gws auth login -s chat`)

**Fetch pattern**:
```bash
# List spaces
gws chat spaces list --format json

# List messages in a space (newest first)
gws chat messages list SPACE_NAME --max-results 50 --format json
```

**Pre-filter**: Skip bot messages, join/leave events, reactions.

**Extract**: Group messages into conversation threads (by time proximity or explicit threads). Extract decisions, action items, knowledge.

**Basalt mapping**:
- Each conversation thread → `communications/{YYYY}/{MM}/comm_{ts}_{hash}.md` with `channel: google_chat`
- Decisions and facts → `knowledge/know_{ts}_{hash}.md`
- New participants → `contacts/{slug}.md`

**Idempotency**: Use message ID as `source_id`.

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
