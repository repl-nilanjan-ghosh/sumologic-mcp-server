# Sumo Logic MCP Server — Release Repo

This repo hosts the distribution and control files for the **Sumo Logic for Replicon** Claude Desktop extension used by the Replicon SME team.

Public because the `.mcpb` is downloaded directly by team members' machines (no GitHub auth). **No credentials or sensitive data should ever be committed here.**

## Repo layout

```
release/
  version.json           ← Checked monthly by all installed extensions
  sumologic-mcp-server.mcpb  ← The actual bundle (4.2 MB, binary)
config/
  blocklist.json         ← Checked daily by all installed extensions
README.md                ← This file
```

## How it works

Every installed extension, on startup:

1. Reads its hardcoded version (e.g. `1.1.0`).
2. If this is the first time starting in a new calendar month, fetches `release/version.json`. If the version there is newer, downloads the `.mcpb` to the user's Desktop.
3. Every day, fetches `config/blocklist.json`. If the current Windows username appears in `blocked_users` (or `block_all` is true), the extension refuses to start.

Both checks fail-open — if GitHub is unreachable, the extension continues to work with the user's current version.

---

## Releasing a new version

1. Update `src/` in the build project (`C:\claude-tools\sumologic-mcp-server`)
2. Bump `CURRENT_VERSION` in `src/index.ts` AND `version` in `manifest.json`. Example: `1.1.0` → `1.1.1` (patch) or `1.2.0` (minor).
3. Build + pack:
   ```cmd
   cd C:\claude-tools\sumologic-mcp-server
   npm run build
   REM ...pack commands (see build notes)
   ```
4. Copy the resulting `.mcpb` into this repo:
   ```cmd
   copy /Y C:\claude-tools\sumologic-mcp-server\sumologic-mcp-server.mcpb C:\claude-tools\sumologic-mcp-server-repo\release\
   ```
5. Update `release/version.json` — set new `version`, `releaseDate`, and a human-readable `releaseNotes` (1-3 sentences).
6. Commit + push:
   ```cmd
   cd C:\claude-tools\sumologic-mcp-server-repo
   git add release/
   git commit -m "Release v1.1.1 — <short summary>"
   git push
   ```
7. Within ~1 month, every team member's extension will detect the new version on startup and download it to their Desktop. They drag it into Claude Desktop Extensions → done.

**Note:** Version comparison uses semver. `1.1.10 > 1.1.9 > 1.1.1 > 1.1.0`. Don't accidentally ship `1.1.01`.

---

## Blocking a user

Edit `config/blocklist.json`:

```json
{
  "blocked_users": ["jane.doe"],
  "block_all": false,
  "message": "Your access is paused pending resolution of <issue>. Contact Nilanjan.",
  "updated_at": "2026-04-24"
}
```

- Usernames are **case-insensitive Windows usernames** (whatever shows in `echo %USERNAME%`).
- `block_all: true` blocks everyone — use only for emergencies (data incident, auth system compromised, etc.).
- Takes effect within 24 hours on every machine. Force-refresh via the `sumo_admin` tool → `block_status` action.
- Blocked extensions **refuse to start** — they appear as failed/errored in Claude Desktop Extensions.

## Unblocking

Remove the username from `blocked_users` and commit. Takes effect within 24 hours.

---

## Initial install (for new team members)

Share this URL — direct `.mcpb` download:

```
https://github.com/repl-nilanjan-ghosh/sumologic-mcp-server/raw/main/release/sumologic-mcp-server.mcpb
```

Team member:
1. Click the link — browser downloads the file
2. Open Claude Desktop → Settings → Extensions → **Advanced settings** → **Install Extension**
3. Select the downloaded `.mcpb`
4. Enter Sumo Logic Access ID + Access Key when prompted
5. Enable the toggle

From that install onwards, monthly updates and blocklist checks are automatic.

---

## Repo security reminders

- **This repo is public.** Don't commit credentials, tokens, or customer data.
- The `.mcpb` file is binary — inspect it before committing (change extension to `.zip` and look inside) if you're unsure.
- `blocklist.json` is public too — don't put customer-identifying info in the block message.
- Keep the commit history clean — if you accidentally commit something sensitive, rewriting history + rotating credentials is required.
