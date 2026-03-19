---
name: roadmap
description: "Generate a comprehensive technical roadmap for building an entire application. Produces a phased delivery plan detailed enough for Claude Code to execute end-to-end: vision, data model, stack decisions, 4-8 phases with feature breakdowns, task checklists, build order, schema evolution, API surface per phase, and 'deliberately not building' scope control. Use after deep-research or when starting a major build. Triggers: 'roadmap', 'plan the build', 'project roadmap', 'delivery plan', 'build plan', 'phase plan', 'plan the whole app', 'end to end plan'."
compatibility: claude-code-only
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
---

# Roadmap

Generate a comprehensive technical roadmap for building an entire application. Detailed enough that Claude Code can pick up any phase and execute it autonomously for hours.

This is not a high-level strategy doc. It's a **delivery blueprint** — every phase has concrete tasks, every task is actionable, and the whole thing is ordered so you can build from phase 1 through to launch without backtracking.

## When to Use

- Starting a major new product (after deep-research, or from a product brief)
- Converting a vague idea into an executable plan
- Planning a multi-week build that will span many Claude Code sessions
- Before saying "build this" — the roadmap is what you hand Claude Code to execute

## Inputs

The skill needs one of these:

| Input | Where to find it |
|-------|-----------------|
| Deep research brief | `.jez/artifacts/research-brief-{topic}.md` (from `/deep-research`) |
| Product brief | User describes what they want to build |
| Existing partial app | Read CLAUDE.md + codebase to understand what exists |
| Competitor to clone/improve | URL or product name — skill analyses it |

If the user just says "plan a note-taking app on Cloudflare", that's enough — ask clarifying questions as needed.

## Workflow

### 1. Establish the Vision

Before any technical planning, nail down:

- **One sentence**: What is this? ("A cloud-native markdown knowledge workspace for teams and AI agents")
- **Who**: Primary users, secondary users, agents? ("Jez first, then Jezweb team, then clients")
- **Why**: What existing tools fail at? What's the gap? ("Brain and Vault are headless black holes — you can't browse them")
- **Constraint**: Stack, budget, timeline, must-haves? ("Cloudflare, must have MCP, needs to be a PWA")
- **Not building**: What's explicitly out of scope? ("No real-time CRDT collab, no plugin ecosystem")

### 2. Define the Stack

Based on the vision and constraints, lock in the technical stack:

```markdown
| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | [framework] | [reason] |
| Backend | [framework + runtime] | [reason] |
| Database | [engine + ORM] | [reason] |
| Auth | [provider] | [reason] |
| Storage | [service] | [reason] |
| Search | [method] | [reason] |
| Hosting | [platform] | [reason] |
```

If a deep-research brief exists, pull the recommendations from there. If not, make opinionated choices based on the user's existing stack (check `~/Documents/` for patterns).

### 3. Design the Data Model

Sketch all tables/collections the full product will need. Not just phase 1 — the complete model. This prevents schema redesigns mid-build.

```markdown
## Data Model

### [entity]
  id, [type]
  [field], [type], [constraints]
  ...
  created_at, updated_at

### [entity]
  ...

### Relationships
- [entity] has many [entity] via [field]
- [entity] belongs to [entity] via [field]
```

Mark which tables are needed in which phase. Phase 1 might only need 3 of 8 tables, but designing them all upfront avoids migration pain.

### 4. Plan the Phases

This is the core of the roadmap. Each phase must:

- **Have a clear goal** — one sentence describing what's different when this phase is done
- **Be independently deployable** — the app works (with reduced features) after each phase
- **Build on the previous phase** — no phase requires ripping out what came before
- **Be completable in 1-3 Claude Code sessions** — if a phase takes more than a day, split it

#### Phase Structure

For each phase:

```markdown
## Phase N — [Name]
*Goal: [One sentence — what the user can do after this phase that they couldn't before]*
*Depends on: Phase N-1*
*Estimated effort: [hours/sessions]*

### What's New
[Bullet list of user-visible features]

### Database Changes
[New tables, new columns, migrations needed]

### API Routes
[New endpoints this phase adds]

### Frontend
[New pages, components, UI changes]

### Infrastructure
[New Cloudflare resources, secrets, config]

### Task Checklist
[Actionable tasks grouped by area — these are what Claude Code executes]

#### Setup
- [ ] [task]

#### Data Layer
- [ ] [task]
- [ ] [task]

#### API
- [ ] [task]

#### Frontend
- [ ] [task]

#### Testing & Polish
- [ ] [task]
- [ ] [task]

### Definition of Done
[How to verify this phase is complete — what to test, what to deploy]
```

### 5. Phase Planning Patterns

#### Phase 1 — Always the MVP

The first phase must produce something **usable by one person for one purpose**. Not a demo, not a skeleton — a working tool that replaces whatever the user currently does (even if it's a spreadsheet or Apple Notes).

Phase 1 scope test: "Would you use this instead of what you use now?" If no, the MVP is too thin.

Typical Phase 1:
- Auth (single user or invite-only)
- Core data model (2-3 tables)
- CRUD for the primary entity
- Basic UI (list + detail + create/edit)
- Deploy to production domain
- Bare minimum styling (dark mode, responsive)

#### Phase 2 — Make It Real

The second phase turns the MVP into something you'd show to others:
- Polish the UI
- Add secondary features (search, filters, sort)
- Better error handling and validation
- Empty states and onboarding (use the `onboarding-ux` skill)
- Mobile responsiveness
- Data export/import

#### Phase 3 — The Differentiator

What makes this product different from alternatives? Build that here:
- AI features, MCP server, semantic search
- The thing competitors don't have
- The reason someone would choose this over the established player

#### Phase 4+ — Growth Features

Each subsequent phase adds capabilities:
- Multi-user / team features
- Advanced views (graph, canvas, calendar, kanban)
- Integrations (API, webhooks, third-party connections)
- Admin / settings
- Performance optimisation
- Public-facing features (sharing, embedding, white-label)

#### Final Phase — Platform

Only if the product is heading toward multi-tenant / SaaS:
- Client workspaces
- Billing / plans
- White-label / custom domains
- API tokens for third-party access

### 6. Build Order

Summarise the phases in a table:

```markdown
| Phase | Goal | New tables | New routes | Sessions |
|-------|------|-----------|-----------|----------|
| 1 | Personal MVP | 3 | 8 | 2-3 |
| 2 | Polish + search | +1 | +4 | 1-2 |
| 3 | AI + MCP | +1 (vectors) | +5 | 2-3 |
| 4 | Team features | +2 | +6 | 2-3 |
| 5 | Integrations | 0 | +4 | 1-2 |
| 6 | Platform | +2 | +8 | 3-4 |
```

### 7. Deliberately Not Building

List what's explicitly out of scope and why. This prevents scope creep during execution:

```markdown
## Deliberately Not Building (v1)

- Real-time collaborative editing (CRDTs) — too complex, Phase 5+ at earliest
- Plugin ecosystem — too much surface area
- Native mobile app — PWA is good enough
- Offline-first with local storage — we're cloud-native
```

### 8. Schema Evolution Map

Show how the database grows across phases:

```markdown
| Table | Phase 1 | Phase 2 | Phase 3 | Phase 4 |
|-------|---------|---------|---------|---------|
| users | ✓ | | | |
| notes | ✓ | +tags col | +embeddings | |
| folders | ✓ | | | |
| note_links | | ✓ | | |
| workspaces | | | | ✓ |
| workspace_users | | | | ✓ |
```

### 9. API Surface Map

Show how the API grows:

```markdown
| Route | Phase | Auth | Purpose |
|-------|-------|------|---------|
| POST /api/auth/* | 1 | — | Authentication |
| GET/POST /api/notes | 1 | Yes | Note CRUD |
| GET /api/search | 2 | Yes | Full-text search |
| GET /api/search/semantic | 3 | Yes | Vector search |
| GET/POST /mcp/tools | 3 | Token | MCP interface |
| POST /api/workspaces | 4 | Yes | Team workspaces |
```

## Output

Write the roadmap to `docs/ROADMAP.md` (or the project root if no `docs/` exists).

The file should be a single markdown document that Claude Code can read at the start of any session and know exactly what to build next. It's the project's north star.

After generating, also update:
- `CLAUDE.md` — add a reference to the roadmap
- `SESSION.md` — set current phase

## Quality Rules

1. **Every task must be actionable** — "Set up auth" is too vague. "Configure better-auth with email/password, create user and session tables, add auth middleware to Hono" is actionable.
2. **Phases must be deployable** — after each phase the app works. No "infrastructure phase" that produces nothing usable.
3. **Phase 1 must be ruthlessly small** — if it takes more than 2-3 sessions, cut scope.
4. **The data model must be complete upfront** — schema redesigns mid-build are the #1 time waster.
5. **"Deliberately not building" is mandatory** — without it, every phase grows.
6. **Tasks are grouped by layer** (data, API, frontend, infra) — Claude Code works in layers, not features.
7. **Each phase has a definition of done** — specific things to test and verify.
8. **Include the stack table** — don't make Claude guess the tech choices per phase.

## Pairing With Other Skills

| Run first | Then this skill | Then |
|-----------|----------------|------|
| `/deep-research deep` | `/roadmap` | Execute phase by phase |
| — (user has a brief) | `/roadmap` | Execute phase by phase |
| `/roadmap` | — | `/fork-discipline document` (if multi-client) |
| `/roadmap` | Phase 1 done | `/ux-audit`, `/onboarding-ux` |
| `/roadmap` | Phase N done | `/project-docs` to update architecture docs |

## Example: How Claude Code Uses This

```
User: "Build phase 3 of the roadmap"

Claude reads docs/ROADMAP.md, finds Phase 3:
- Goal: AI search + MCP server
- Tasks: 15 checklist items
- New tables: notes_embeddings (Vectorize)
- New routes: /api/search/semantic, /mcp/tools
- Definition of done: semantic search returns relevant results, MCP tools accessible via API token

Claude executes all 15 tasks, deploys, tests against the definition of done, updates SESSION.md.
```

That's the target: a roadmap so detailed that "build phase 3" is a complete instruction.
