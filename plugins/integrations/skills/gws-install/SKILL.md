---
name: gws-install
description: >
  Quick install of the Google Workspace CLI (gws) on an additional machine using existing
  OAuth credentials. Requires client_secret.json from a previous gws-setup.
  Use when setting up gws on a new computer, reinstalling after a fresh OS, or
  configuring a second workstation. Triggers: "install gws", "gws on new machine",
  "gws install", "set up gws again".
compatibility: claude-code-only
---

# Google Workspace CLI — Quick Install

Install `gws` on an additional machine using OAuth credentials from a previous setup. Produces an authenticated CLI with all agent skills ready to use.

**Prerequisite**: The user must have `client_secret.json` from a previous `gws-setup` (or from Google Cloud Console). If they don't have it, use the `gws-setup` skill instead.

## Workflow

### Step 1: Pre-flight Checks

```bash
which gws && gws --version
ls ~/.config/gws/client_secret.json
gws auth status
```

If already authenticated with the right scopes, skip to Step 4.

### Step 2: Install the CLI

```bash
npm install -g @googleworkspace/cli
gws --version
```

### Step 3: Set Up Credentials

Ask the user to provide their `client_secret.json`. Three options:

**Option A — Paste the JSON content:**

Ask the user to paste the JSON. Write it to `~/.config/gws/client_secret.json`:

```bash
mkdir -p ~/.config/gws
```

Expected format:

```json
{
  "installed": {
    "client_id": "...",
    "project_id": "...",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "client_secret": "...",
    "redirect_uris": ["http://localhost"]
  }
}
```

**Option B — File path:**

If the user has the file locally (e.g. in Downloads):

```bash
mkdir -p ~/.config/gws
cp /path/to/client_secret.json ~/.config/gws/client_secret.json
```

**Option C — Environment variables:**

```bash
export GOOGLE_WORKSPACE_CLI_CLIENT_ID="your-client-id"
export GOOGLE_WORKSPACE_CLI_CLIENT_SECRET="your-client-secret"
```

### Step 4: Authenticate

Ask which Google account to use, then run:

```bash
gws auth login --full
```

Or with specific services to match the original setup:

```bash
gws auth login -s gmail,drive,calendar,sheets,docs,chat,tasks
```

The user can check their original machine's scopes with `gws auth status` to see what was granted.

After browser approval, verify:

```bash
gws auth status
```

### Step 5: Install Agent Skills

```bash
npx skills add googleworkspace/cli -g --agent claude-code --all
```

This installs 90+ skills into `~/.claude/skills/`. Safe to re-run if skills are already installed.

### Step 6: Verify

```bash
gws auth status
gws calendar +agenda --today
gws gmail +triage
```

---

## Troubleshooting

### "Auth error — credentials missing or invalid" (exit code 2)

- Check `~/.config/gws/client_secret.json` exists and has valid JSON
- Re-run `gws auth login`

### Token expired

- If the GCP app is in "Testing" status, tokens expire after 7 days
- Re-run `gws auth login` to refresh
- For permanent tokens, push the app to Production in GCP Console OAuth consent screen

### Skills not appearing in Claude Code

- Skills load at session start — restart Claude Code after installing
- Verify: `ls ~/.claude/skills/gws-* | wc -l` should show 30+ directories

## See Also

- [gws-setup](../gws-setup/SKILL.md) — First-time setup including GCP project creation
- [gws-shared](~/.claude/skills/gws-shared/SKILL.md) — Auth patterns and global flags
