# Sync Patterns

How `~/.cortex/` Basalt files sync to external systems. Files are the source of truth. Everything else is a projection.

**Status**: Not yet built. Reference when implementing cortex-sync mode.

---

## Architecture

```
~/.cortex/               <- source of truth (Basalt markdown files)
      |
      +-> Frond API      <- visual web interface, search, browse
      +-> D1             <- SQL queries, structured lookups
      +-> Vectorize      <- semantic search via embeddings
```

One-way sync. Files are truth. External systems are read projections.

---

## Frond Sync

Push Basalt files to Frond as notes via its API.

### Prerequisites
- Frond API token (generate at frond.au settings)
- Store as `FROND_API_TOKEN` in `~/.cortex/.env`

### Mapping

| Basalt field | Frond field |
|-------------|-------------|
| `id` | `external_id` (upsert key) |
| Full file content | `content` |
| First `# heading` | `title` (auto-extracted by Frond) |
| `type` | Determines target folder |
| `updated` | Compare to skip unchanged |

### Folder mapping

| Cortex path | Frond folder |
|-------------|-------------|
| `clients/` | Clients |
| `contacts/` | Contacts |
| `communications/` | Communications |
| `knowledge/` | Knowledge |
| `notes/` | Notes |

### Sync logic

```python
for md_file in walk_cortex():
    basalt = parse_frontmatter(md_file)
    existing = frond_find_by_external_id(basalt['id'])

    if existing and basalt['updated'] <= existing['synced_at']:
        continue  # unchanged

    if existing:
        frond_patch(existing['id'], content=md_file.read_text())
    else:
        folder_id = get_or_create_folder(basalt['type'])
        frond_create(folder_id=folder_id, content=md_file.read_text())

    update_sync_state(basalt['id'], synced_at=now())
```

---

## D1 Sync

Parse frontmatter, map fields to columns, upsert.

### Mapping

| Frontmatter | D1 column | Notes |
|-------------|-----------|-------|
| `id` | PRIMARY KEY | Upsert key |
| `type` | Routes to table | `client` -> `clients` etc |
| `created` | `created_at` | ISO 8601 |
| `updated` | `updated_at` | Drives incremental sync |
| `summary` | `summary` | Vectorize embedding input |
| `tags` | `tags` | JSON array |
| `source` | `source` | |
| `source_id` | `source_id` | Idempotency |

Upsert: `INSERT OR REPLACE` keyed on `id`. Use `updated` to skip unchanged.

---

## Vectorize Sync

Embed the `summary` field. Store vector with metadata for filtering without D1.

### Embedding input
Use only the `summary` field (1-3 sentences, dense with names/context).

### Metadata payload
```json
{
  "id": "client_smithholdings.com.au",
  "type": "client",
  "client_domain": "smithholdings.com.au",
  "visibility": "private",
  "updated": "2026-03-13T06:15:00+11:00"
}
```

### Pattern
```python
embedding = workers_ai_embed(basalt['summary'])
vectorize.upsert([{
    "id": basalt['id'],
    "values": embedding,
    "metadata": { ... }
}])
```

---

## Sync State

Track sync progress in `state.json`:

```json
{
  "frond_sync": {
    "client_smithholdings.com.au": {
      "frond_note_id": "abc123",
      "synced_at": "2026-03-13T06:15:00+11:00"
    }
  }
}
```

---

## Running

```bash
# Future CLI patterns:
python3 cortex-sync.py --target frond
python3 cortex-sync.py --target d1
python3 cortex-sync.py --target vectorize
python3 cortex-sync.py  # all targets
```
