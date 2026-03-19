# Basalt Format

**Basalt** is the file format standard used by Knowledge Cortex.

Name: basalt is the volcanic rock that obsidian forms from. It's the underlying substrate — fully compatible with Obsidian but not dependent on it.

## What It Is

Markdown files with structured YAML frontmatter. Every file is:
- A valid Obsidian note (open `~/.cortex/` as an Obsidian vault)
- Readable by Claude Code directly without parsing
- Syncable to D1 via frontmatter field mapping
- Embeddable in Vectorize via the `summary` field
- Importable into Frond via `/api/notes`

Files are the source of truth. D1, Vectorize, and Frond are projections.

---

## Folder Structure

```
~/Documents/basalt-cortex/        # vault root — syncs to basaltcortex.com via CLI daemon
├── .obsidian/                    # Obsidian config (can be empty or gitignored)
├── clients/
│   └── smithholdings.com.au.md
├── contacts/
│   └── jane-smith.md
├── communications/
│   └── 2026/03/
│       └── 2026-03-10-website-redesign-quote.md
├── knowledge/
│   └── smith-holdings-prefers-phone.md
├── projects/                     # future
├── notes/                        # manual notes, not mined
└── state.json                    # cursor + run history (not a note)
```

---

## Universal Frontmatter Fields

Every Basalt note has these fields:

```yaml
---
# Identity
id: client_smithholdings.com.au       # stable, deterministic, machine-generated
type: client                           # client | contact | communication | knowledge | project | note
created: 2024-03-15T09:00:00+11:00
updated: 2026-03-13T06:15:00+11:00

# Obsidian-native (Obsidian reads these, ignores everything else)
tags: [client, active]
aliases: [Smith Holdings, SH]

# Provenance
source: gmail                          # gmail | google_chat | calendar | manual | frond
source_id: 18e2a3b4c5d6e7f8           # idempotency key — check before processing
source_url: https://mail.google.com/mail/u/0/#all/18e2a3b4c5d6e7f8

# Semantic layer
summary: "Web dev client since 2019. Prefers phone. Active WordPress maintenance plan."
# ^ 1-3 sentences, dense with names/context. This is the Vectorize embedding input.

# Access
visibility: private                    # private | team | public
---
```

### `id` generation rules

The `id` field is for machine dedup/sync. It stays deterministic and unique.

| Type | ID Pattern | Example |
|------|---------|---------|
| client | `client_{domain}` | `client_smithholdings.com.au` |
| contact | `contact_{email_slug}` | `contact_jane-smith-smithholdings.com.au` |
| communication | `comm_{unix_ts}_{4char_hash}` | `comm_1741824000_a3f2` |
| knowledge | `know_{unix_ts}_{4char_hash}` | `know_1741824000_b7c1` |
| project | `project_{slug}` | `project_smith-redesign-2025` |

### Filename conventions (human-readable)

Filenames are what users see in the sidebar. They should be scannable and descriptive. The `id` in frontmatter handles dedup — the filename is for humans.

| Type | Filename Pattern | Example |
|------|-----------------|---------|
| client | `{domain}.md` | `smithholdings.com.au.md` |
| contact | `{first-last}.md` | `jane-smith.md` |
| communication | `{YYYY-MM-DD}-{subject-slug}.md` | `2026-03-10-website-redesign-quote.md` |
| knowledge | `{topic-slug}.md` | `smith-holdings-prefers-phone.md` |
| project | `{project-slug}.md` | `smith-redesign-2025.md` |

**Rules:**
- Slugify: lowercase, hyphens, no special characters
- Communications: always lead with ISO date for chronological sorting
- Knowledge: lead with the subject/client, describe the fact
- Keep slugs under ~60 chars — long enough to be descriptive, short enough to scan

---

## Type: `client`

File: `clients/{domain}.md`

```yaml
---
id: client_smithholdings.com.au
type: client
created: 2024-03-15T09:00:00+11:00
updated: 2026-03-13T06:15:00+11:00
tags: [client, active]
aliases: [Smith Holdings, SH]
source: gmail
source_id: ~
source_url: ~
summary: "Web dev client since 2019. Prefers phone. Active WordPress maintenance plan."
visibility: private

name: Smith Holdings Pty Ltd
domain: smithholdings.com.au
abn: ~
industry: ~
first_seen: 2024-03-15
last_seen: 2026-03-10
thread_count: 12
significance: 3                        # 1=minimal 2=light 3=normal 4=important 5=critical

related: ["[[contacts/jane-smith]]"]
---

# Smith Holdings Pty Ltd

## Contacts
- [[contacts/jane-smith]] — Director

## Recent Communications
<!-- cortex-mine appends here -->

## Facts
<!-- cortex-mine appends here -->
```

---

## Type: `contact`

File: `contacts/{slug}.md`

```yaml
---
id: contact_jane-smith-smithholdings.com.au
type: contact
created: 2024-03-15T09:00:00+11:00
updated: 2026-03-10T14:22:00+11:00
tags: [contact]
aliases: [Jane Smith]
source: gmail
source_id: ~
source_url: ~
summary: "Director at Smith Holdings. Primary contact for all web projects. Prefers brief emails."
visibility: private

name: Jane Smith
email: person@smithholdings.com.au
phone: ~
role: Director
company: Smith Holdings Pty Ltd
company_domain: smithholdings.com.au
first_seen: 2024-03-15
last_seen: 2026-03-10
thread_count: 7

related: ["[[clients/smithholdings.com.au]]"]
---

# Jane Smith
```

---

## Type: `communication`

File: `communications/{YYYY}/{MM}/{YYYY-MM-DD}-{subject-slug}.md`

Communications use a **universal header + freeform semantic fields** pattern. The universal fields are always present. Freeform fields are added only when evidenced by the thread content — see [field-catalog.md](field-catalog.md) for the full vocabulary.

### Simple thread (2 messages):

```yaml
---
id: comm_1742352831_nova
type: communication
channel: email
subject: "Re: Nova tinting website"
date: 2026-03-19T14:53:51+11:00
direction: inbound
participants:
  - name: Brad Kendall
    email: novapitstop@gmail.com
  - name: Jeremy Dawes
    email: jeremy@jezweb.au
message_count: 2
summary: "Brad Kendall from Nova Pitstop confirmed 1pm meeting at Jezweb office to discuss tinting website. Also trades as Hi-Tech Autoglass Solutions."
tags: [meeting, website-project, automotive]
source: gmail
source_id: 19cade5e9cf2fbc6

# Freeform — only what's in the content
companies: [Nova Pitstop, Hi-Tech Autoglass Solutions]
phone: "02 4954 6828"
website: www.novapitstop.com.au/book-online
meeting_time: "1pm"
meeting_location: "Jeremy's office"
primary_intent: scheduling
---

# Nova tinting website — 19 Mar 2026

Brad Kendall confirmed 1pm meeting at Jezweb office to discuss the tinting website project. Also trades as Hi-Tech Autoglass Solutions.
```

### Complex thread (8 messages):

```yaml
---
id: comm_1740182400_skin
type: communication
channel: email
subject: Micro Skin Website
date: 2026-02-19T00:00:00+10:00
direction: bidirectional
participants:
  - name: Justin
    email: Justin@bigcolour.com.au
    company: Big Colour
  - name: Belinda
    email: Belinda@bigcolour.com.au
    company: Big Colour
  - name: Jeremy Dawes
    email: jeremy@jezweb.au
    company: Jezweb
  - name: Marianne Boughton
    email: marianne@jezweb.net
    company: Jezweb
    role: accounts
  - name: Karen Irvine
    email: karen@jezweb.net
    company: Jezweb
message_count: 8
summary: "Big Colour commissioning Micro Skin product website. Justin requesting design iterations including darker grey and white outline changes. Multi-week back-and-forth."
tags: [website-design, design-feedback, micro-skin]
source: gmail
source_id: 19c972f9c508ab2f

# Freeform — richer because thread is richer
companies: [Big Colour, Jezweb]
project: micro-skin-website
product: Micro Skin
primary_intent: project_feedback
client_lifecycle_stage: active_project
design_changes: [darker grey, white outline adjustment]
business_relationship: client-vendor
---

# Micro Skin Website — 19 Feb to 3 Mar 2026

Big Colour (signage company) working with Jezweb on a website for the Micro Skin product line. Justin from Big Colour drove the design feedback, requesting specific changes including making the grey darker and adjusting the white outline. Jeremy forwarded feedback internally to Marianne and Karen. Multiple rounds of iteration over two weeks as the design was refined to Big Colour's requirements.
```

### Key principles:

- **Summary length is proportional** to thread complexity
- **Freeform fields vary** per thread — a bug report looks different from a pricing dispute
- **No empty fields** — if it's not in the content, don't add it
- See [field-catalog.md](field-catalog.md) for the full vocabulary of discoverable fields
```

---

## Type: `knowledge`

File: `knowledge/{topic-slug}.md`

```yaml
---
id: know_1741824000_b7c1
type: knowledge
created: 2026-03-10T11:30:00+11:00
updated: 2026-03-10T11:30:00+11:00
tags: [knowledge, preference]
source: gmail
source_id: 18e2a3b4c5d6e7f8
source_url: https://mail.google.com/mail/u/0/#all/18e2a3b4c5d6e7f8
summary: "Jane Smith at Smith Holdings prefers phone contact over email for project discussions."
visibility: private

kind: preference                       # decision | commitment | deadline | amount | preference | feedback | relationship | other
client_domain: smithholdings.com.au
contact_email: person@smithholdings.com.au
project: ~
thread_date: 2026-03-10T11:30:00+11:00

related: ["[[clients/smithholdings.com.au]]", "[[contacts/jane-smith]]"]
---

Client prefers phone contact over email for project discussions.
```

---

## D1 Sync Mapping

When syncing to D1, extract frontmatter fields directly:

| Frontmatter | D1 column | Notes |
|-------------|-----------|-------|
| `id` | PRIMARY KEY | Upsert key |
| `type` | routes to table | `client` → `clients` etc |
| `created` | `created_at` | ISO 8601 |
| `updated` | `updated_at` | Drives incremental sync |
| `summary` | `summary` | Vectorize embedding input |
| `visibility` | `visibility` | Future team/public access |
| `tags` | `tags` | JSON array |
| `source` | `source` | |
| `source_id` | `source_id` | Idempotency check |

Upsert pattern: `INSERT OR REPLACE` keyed on `id`. Use `updated` to skip unchanged records.

---

## Vectorize Embedding

Embed the `summary` field only. Keep summary:
- 1–3 sentences max
- Dense with names, domains, context
- A factual description, not a heading or label

Vector metadata payload: `id`, `type`, `client_domain`, `visibility`, `updated`
(lets you filter without a D1 roundtrip)

---

## Obsidian Compatibility Notes

- `tags` and `aliases` — Obsidian reads and renders natively
- All other fields — Obsidian ignores silently, no errors or warnings
- `[[wiki links]]` in `related` — Obsidian renders as clickable cross-references
- Dataview plugin can query any frontmatter field as a table or list, zero config
- Open `~/.cortex/` as a vault in Obsidian and it works today
