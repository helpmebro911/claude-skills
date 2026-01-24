# Claude Session Observer - Planning Document

**Project**: claude-session-observer
**Status**: Planning
**Created**: 2026-01-24
**Author**: Jeremy Dawes (Jez)

---

## Vision

A standalone desktop application that watches Claude Code sessions and provides:

1. **Real-time session tracking** - See all active Claude Code sessions at a glance
2. **Task progress visualization** - Todo lists, completion status, context usage
3. **AI-powered summaries** - Periodic intelligent explanations of what's happening
4. **Context suggestions** - Surface useful details worth saving to project docs
5. **Session history** - Record of what was accomplished across sessions

### The Problem

When working with Claude Code, valuable information scrolls past:
- Decisions made and their rationale
- Gotchas discovered during implementation
- Patterns established that should be documented
- Errors encountered and how they were resolved

This information is ephemeral - lost when the session ends or scrolls out of view. The Session Observer captures, summarizes, and surfaces this knowledge.

---

## Target Users

- Developers using Claude Code CLI for daily work
- Teams wanting visibility into AI-assisted development
- Anyone who wants to capture institutional knowledge from AI sessions

### Platforms

- **Primary**: macOS, Linux (where Claude Code runs)
- **Secondary**: Windows (if demand exists)

---

## Core Features

### 1. Multi-Session Dashboard

```
┌─────────────────────────────────────────────────────────────┐
│ Claude Session Observer                              ─ □ × │
├─────────────────────────────────────────────────────────────┤
│ 📂 claude-skills (active)                    Context: 47%  │
│ ├─ ✅ Research existing statusline tools                   │
│ ├─ 🔄 Design session observer architecture                 │
│ ├─ ⏳ Create hook to emit session events                   │
│ └─ ⏳ Build floating widget UI                             │
│                                                             │
│ 💬 Latest Summary (2 min ago)                              │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ Explored the existing ContextBricks statusline and      ││
│ │ discussed options for a standalone progress widget.     ││
│ │ User wants AI-powered summaries of session activity.    ││
│ │                                                         ││
│ │ 💡 Worth saving: User prefers Electron for cross-       ││
│ │ platform. Consider adding to project preferences.       ││
│ └─────────────────────────────────────────────────────────┘│
│                                                             │
│ 📂 website-project (idle - 15m)              Context: 23%  │
│ └─ ✅ All 3 tasks completed                                │
├─────────────────────────────────────────────────────────────┤
│ 🔔 Notifications                                           │
│ • claude-skills: Completed "Research statusline tools"     │
│ • website-project: Session idle for 15 minutes             │
└─────────────────────────────────────────────────────────────┘
```

**Shows per session:**
- Project name/directory
- Current todo list with status (✅ completed, 🔄 in progress, ⏳ pending)
- Context window usage (percentage + visual bar)
- Session duration
- Active/idle status
- Latest AI summary

### 2. AI-Powered Summaries

Periodic summaries generated at key moments:

**Triggers:**
- Task completed
- Meaningful milestone (commit, file creation)
- Time-based (every 10-15 min of active work)
- Context threshold (80%+ usage)
- Error resolution
- Session end/pause

**Summary Content:**
- What just happened (1-3 sentences)
- Key decisions made
- Files affected
- "Worth saving" suggestions

### 3. "Worth Saving" Detection

AI identifies information that should be persisted:

**Categories:**
- **Decisions**: "Chose Hono over Express because of Cloudflare Workers compatibility"
- **Gotchas**: "D1 doesn't support JOIN on subqueries - use multiple queries"
- **Patterns**: "Established repository pattern for all data access"
- **Preferences**: "User prefers explicit error handling over try-catch"
- **Configuration**: "Set up with TypeScript strict mode enabled"

**Actions:**
- Show in UI with context
- "Copy to clipboard" button
- "Add to CLAUDE.md" button (opens PR or appends to file)
- Auto-collect into session summary

### 4. Session History

Persistent record of past sessions:

- Browse by project/date
- Search across sessions
- See what was accomplished
- Review extracted insights
- Export session reports

### 5. Notifications

Non-intrusive alerts for:

- Task completions
- Session milestones
- Errors requiring attention
- Context running low
- "Worth saving" suggestions
- Session idle warnings

---

## Architecture

### System Overview

```
┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│  Claude Code     │────▶│  Event Service  │────▶│  Observer App    │
│  Sessions        │     │  (background)   │     │  (Electron)      │
│  (with hooks)    │     │                 │     │                  │
└──────────────────┘     └────────┬────────┘     └──────────────────┘
                                  │
                         ┌────────┴────────┐
                         │                 │
                         ▼                 ▼
                  ┌─────────────┐   ┌─────────────┐
                  │  SQLite DB  │   │ AI Service  │
                  │  (events,   │   │ (summaries) │
                  │   history)  │   │             │
                  └─────────────┘   └─────────────┘
```

### Components

#### 1. Claude Code Hooks

Claude Code supports hooks that run on specific events. We'll create hooks that emit session data.

**Hook Events to Capture:**
- `on_tool_call` - Tool invocations (Read, Write, Edit, Bash, etc.)
- `on_todo_update` - Todo list changes
- `on_message` - Claude's responses (for summarization)
- `on_session_start` / `on_session_end`

**Hook Output Format:**
```json
{
  "event_type": "todo_update",
  "session_id": "abc123",
  "project_dir": "/home/user/my-project",
  "timestamp": "2026-01-24T10:30:00Z",
  "data": {
    "todos": [...],
    "changed": { "task": "Build API endpoint", "status": "completed" }
  }
}
```

**Delivery Mechanism:**
- Option A: Write to file (`~/.claude-observer/events.jsonl`)
- Option B: HTTP POST to local server (`http://localhost:9847/events`)
- Option C: Unix socket (`/tmp/claude-observer.sock`)

Recommend: **File-based** for simplicity, with inotify/fswatch for real-time updates.

#### 2. Event Service (Background Process)

Runs as a daemon/background service:

**Responsibilities:**
- Watch for new events from hooks
- Parse and validate events
- Store in SQLite database
- Trigger AI summarization at appropriate moments
- Expose API for Observer App

**Tech Options:**
- Node.js with chokidar (file watching)
- Go binary (lightweight, easy to daemonize)
- Rust (performant, single binary)
- Python with watchdog

Recommend: **Node.js** for ecosystem compatibility with Electron.

#### 3. SQLite Database

Simple local storage:

**Tables:**
```sql
-- Active and historical sessions
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  project_dir TEXT,
  project_name TEXT,
  started_at DATETIME,
  ended_at DATETIME,
  status TEXT  -- 'active', 'idle', 'ended'
);

-- Raw events from hooks
CREATE TABLE events (
  id INTEGER PRIMARY KEY,
  session_id TEXT,
  event_type TEXT,
  timestamp DATETIME,
  data JSON,
  FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- AI-generated summaries
CREATE TABLE summaries (
  id INTEGER PRIMARY KEY,
  session_id TEXT,
  timestamp DATETIME,
  trigger TEXT,  -- 'task_complete', 'time_based', 'milestone', etc.
  content TEXT,
  worth_saving JSON,  -- Array of suggested items
  FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Saved insights (user confirmed "worth saving")
CREATE TABLE insights (
  id INTEGER PRIMARY KEY,
  session_id TEXT,
  category TEXT,  -- 'decision', 'gotcha', 'pattern', 'preference'
  content TEXT,
  saved_to TEXT,  -- Where it was saved (CLAUDE.md path, etc.)
  created_at DATETIME,
  FOREIGN KEY (session_id) REFERENCES sessions(id)
);

-- Todo snapshots for tracking progress
CREATE TABLE todos (
  id INTEGER PRIMARY KEY,
  session_id TEXT,
  timestamp DATETIME,
  todos JSON,
  FOREIGN KEY (session_id) REFERENCES sessions(id)
);
```

#### 4. AI Summarization Service

Generates intelligent summaries from event streams.

**Model Options:**
- **Gemini Flash 2.0** - Fast, cheap, good for summaries
- **Claude Haiku** - Consistent with Claude Code experience
- **Local LLM** (Ollama) - Privacy, no API costs
- **OpenAI GPT-4o-mini** - Fast, cheap alternative

Recommend: **Configurable**, default to Gemini Flash for speed/cost.

**Summarization Prompt Template:**
```
You are summarizing a Claude Code session for the developer.

Recent activity:
{event_stream}

Current todos:
{todo_list}

Generate:
1. A 1-3 sentence summary of what just happened
2. Any key decisions or patterns established
3. "Worth saving" items - insights that should be documented

Keep it concise and actionable.
```

#### 5. Observer App (Electron)

Desktop application with floating widget capability.

**Features:**
- Always-on-top floating window (optional)
- System tray icon with quick status
- Full dashboard view
- Settings/preferences
- Session history browser

**Tech Stack:**
- Electron (cross-platform)
- React + TypeScript (UI)
- Tailwind CSS (styling)
- Zustand (state management)
- TanStack Query (data fetching from Event Service)

**Window Modes:**
1. **Compact** - Small floating widget showing active session + progress
2. **Expanded** - Full dashboard with all sessions
3. **Tray only** - Minimized to system tray

---

## Implementation Phases

### Phase 1: Foundation (MVP)

**Goal:** Basic session tracking with todo progress

**Deliverables:**
1. Claude Code hook that emits events to file
2. Event service that watches file and stores in SQLite
3. Simple Electron app showing active sessions + todos
4. Basic context usage display

**No AI yet** - just event capture and display.

**Estimated scope:** Core infrastructure

### Phase 2: AI Summaries

**Goal:** Intelligent periodic summaries

**Deliverables:**
1. AI service integration (Gemini Flash default)
2. Summary generation on key triggers
3. Display summaries in UI
4. Summary history per session

**Estimated scope:** AI integration layer

### Phase 3: Worth Saving

**Goal:** Surface and persist valuable insights

**Deliverables:**
1. "Worth saving" detection in summaries
2. UI for reviewing suggestions
3. Actions: copy, save to file, dismiss
4. Insights database and history

**Estimated scope:** Feature enhancement

### Phase 4: Polish & History

**Goal:** Full-featured production app

**Deliverables:**
1. Session history browser
2. Search across sessions
3. Export/reporting
4. Notifications system
5. Settings/preferences UI
6. Auto-update mechanism

**Estimated scope:** Production polish

### Phase 5: Advanced Features (Future)

**Ideas:**
- Team sharing (sync insights across team)
- Analytics dashboard (patterns across sessions)
- Integration with project management tools
- Voice summaries (TTS)
- Mobile companion app

---

## Technical Decisions to Make

### 1. Event Delivery Mechanism

| Option | Pros | Cons |
|--------|------|------|
| File (JSONL) | Simple, reliable, debuggable | Polling or file watch needed |
| HTTP POST | Real-time, standard | Requires server running |
| Unix Socket | Fast, real-time | Platform-specific |
| SQLite direct | No middleman | Locking concerns |

**Recommendation:** Start with JSONL file + fswatch, migrate to socket if needed.

### 2. Event Service Architecture

| Option | Pros | Cons |
|--------|------|------|
| Embedded in Electron | Single process | Electron must run |
| Separate Node daemon | Can run headless | Two processes to manage |
| Go/Rust binary | Lightweight, fast | Different language |

**Recommendation:** Separate Node daemon (can run without UI, supports headless servers).

### 3. AI Provider

| Option | Pros | Cons |
|--------|------|------|
| Gemini Flash | Fast, cheap ($0.075/1M) | Requires API key |
| Claude Haiku | Consistent experience | More expensive |
| Local (Ollama) | Free, private | Requires setup, slower |
| OpenAI GPT-4o-mini | Fast, reliable | Requires API key |

**Recommendation:** Support multiple providers, default to Gemini Flash.

### 4. UI Framework

| Option | Pros | Cons |
|--------|------|------|
| Electron + React | Rich ecosystem | Heavy (100MB+) |
| Tauri + React | Lightweight (10MB) | Less mature |
| Native (Swift/GTK) | Most native feel | Platform-specific |

**Recommendation:** Electron for v1 (faster development), consider Tauri for v2.

---

## Claude Code Hook Implementation

### Hook Configuration

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "on_tool_call": {
      "command": "~/.claude-observer/hooks/on-tool-call.sh"
    },
    "on_message": {
      "command": "~/.claude-observer/hooks/on-message.sh"
    }
  }
}
```

### Hook Script Example

```bash
#!/bin/bash
# ~/.claude-observer/hooks/on-tool-call.sh

# Read event data from stdin
EVENT_DATA=$(cat)

# Add metadata
ENRICHED=$(echo "$EVENT_DATA" | jq --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  '. + {timestamp: $ts, session_id: env.CLAUDE_SESSION_ID}')

# Append to events file
echo "$ENRICHED" >> ~/.claude-observer/events.jsonl
```

### Events to Capture

**High Priority:**
- `TodoWrite` calls (todo list changes)
- `Write` / `Edit` calls (files modified)
- `Bash` calls with git commands (commits, pushes)
- Session start/end

**Medium Priority:**
- `Read` calls (files being explored)
- `Glob` / `Grep` calls (search patterns)
- Error responses

**Low Priority (for summaries):**
- All tool calls (for activity stream)
- Claude's text responses (for context)

---

## Directory Structure

```
claude-session-observer/
├── README.md
├── LICENSE
├── package.json
│
├── apps/
│   ├── electron/              # Electron desktop app
│   │   ├── src/
│   │   │   ├── main/          # Main process
│   │   │   ├── renderer/      # React UI
│   │   │   └── preload/
│   │   ├── package.json
│   │   └── electron-builder.json
│   │
│   └── daemon/                # Background event service
│       ├── src/
│       │   ├── watcher.ts     # File/event watcher
│       │   ├── db.ts          # SQLite operations
│       │   ├── ai.ts          # AI summarization
│       │   └── api.ts         # HTTP API for Electron
│       └── package.json
│
├── packages/
│   ├── hooks/                 # Claude Code hooks
│   │   ├── on-tool-call.sh
│   │   ├── on-message.sh
│   │   └── install.sh
│   │
│   ├── shared/                # Shared types/utils
│   │   ├── types.ts
│   │   └── constants.ts
│   │
│   └── ui-components/         # Shared React components
│       └── src/
│
├── scripts/
│   ├── install.sh             # One-line installer
│   ├── uninstall.sh
│   └── dev.sh
│
└── docs/
    ├── SETUP.md
    ├── HOOKS.md
    └── API.md
```

---

## Configuration

### User Settings

`~/.claude-observer/config.json`:

```json
{
  "ai": {
    "provider": "gemini",  // "gemini", "claude", "openai", "ollama"
    "api_key_env": "GEMINI_API_KEY",
    "model": "gemini-2.0-flash"
  },
  "summaries": {
    "time_interval_minutes": 15,
    "on_task_complete": true,
    "on_milestone": true,
    "context_threshold_percent": 80
  },
  "ui": {
    "start_minimized": false,
    "always_on_top": false,
    "theme": "system",  // "light", "dark", "system"
    "compact_mode": false
  },
  "notifications": {
    "enabled": true,
    "sound": false,
    "task_complete": true,
    "worth_saving": true
  }
}
```

---

## Open Questions

1. **How to get session ID?** Does Claude Code expose a session identifier via environment variable or API?

2. **Hook reliability** - What happens if hook scripts fail? Does Claude Code continue?

3. **Multi-machine sync** - Should insights sync across machines? (Future consideration)

4. **Privacy** - How to handle sensitive data in events? Configurable redaction?

5. **Performance** - How to avoid impacting Claude Code performance with hooks?

6. **Context access** - Can hooks access Claude's conversation context, or only tool call data?

---

## Success Metrics

1. **Capture Rate** - % of meaningful events captured from sessions
2. **Summary Quality** - User satisfaction with AI summaries (thumbs up/down)
3. **Insights Saved** - Number of "worth saving" items actually saved
4. **Session Coverage** - % of Claude Code sessions being tracked
5. **Performance Impact** - Latency added by hooks (target: <50ms)

---

## Getting Started (For Implementation)

### Prerequisites

- Node.js 20+
- pnpm (recommended) or npm
- Claude Code installed and configured
- API key for AI provider (Gemini, OpenAI, or Claude)

### Bootstrap Commands

```bash
# Create repo
mkdir claude-session-observer && cd claude-session-observer
git init

# Initialize monorepo
pnpm init
# Add workspace config for apps/ and packages/

# Create Electron app
cd apps && pnpm create electron-app electron -- --template=vite-react-ts

# Create daemon service
mkdir daemon && cd daemon && pnpm init

# Install hooks
mkdir -p ~/.claude-observer/hooks
cp packages/hooks/* ~/.claude-observer/hooks/
chmod +x ~/.claude-observer/hooks/*.sh
```

### Development Workflow

1. Start daemon: `pnpm --filter daemon dev`
2. Start Electron: `pnpm --filter electron dev`
3. Run Claude Code in a test project
4. Events should flow through the system

---

## References

- [Claude Code Hooks Documentation](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Claude Code Status API](https://docs.anthropic.com/en/docs/claude-code/statusline)
- [Electron Documentation](https://www.electronjs.org/docs)
- [Gemini API](https://ai.google.dev/docs)
- [better-sqlite3](https://github.com/WiseLibs/better-sqlite3)
- [chokidar](https://github.com/paulmillr/chokidar) (file watching)

---

## Related Work

- **ContextBricks** (claude-skills/tools/statusline) - In-terminal status display
- **Claude Code MCP** - Model Context Protocol integration
- **Session management** - claude-skills/skills/project-session-management

---

*This document captures the vision for Claude Session Observer. Start with Phase 1 (MVP) and iterate based on real usage.*
