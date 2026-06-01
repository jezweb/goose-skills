---
name: install-gws
version: 1.0.0
description: "Install the official Google Workspace CLI (gws) so the agent can drive Gmail, Drive, Docs, Sheets, Calendar, Chat and Admin from the shell. Use when the user wants Google Workspace access on a box, asks to 'install gws' / 'add Google Workspace', or a workflow needs to read mail / write a Doc / update a Sheet. gws is a shell CLI, not an MCP server — the agent calls it through its developer/shell tool."
metadata:
  category: "setup"
  homepage: "https://github.com/googleworkspace/cli"
  installs-bin: "gws"
---

# Install gws (Google Workspace CLI)

Installs [`@googleworkspace/cli`](https://github.com/googleworkspace/cli) — Google's official
Workspace command-line tool. Once installed and authenticated, the agent reads/sends Gmail,
manages Drive, writes Docs and Sheets, handles Calendar/Chat/Tasks, and (with an admin account)
pulls audit reports — all through the shell.

Unlike OfficeCLI, gws is **not an MCP server**. There's nothing to register in
`config.yaml`; the agent uses it via its existing `developer`/shell tool (`gws <service>
<resource> <method>`). This skill just gets the binary on PATH and authenticated.

## Steps

### 1. Install the binary (needs Node ≥ 18)

```bash
node -v   # confirm Node is present
npm install -g @googleworkspace/cli
gws --version    # prints e.g. "gws 0.13.2"
```

If `npm i -g` hits an EACCES permission error, either use a Node managed by `nvm`/`fnm`
(no sudo needed) or set a user prefix (`npm config set prefix ~/.local` and put
`~/.local/bin` on PATH). Don't `sudo npm -g` on a shared box.

### 2. Authenticate (hand the OAuth step to the human)

```bash
gws auth login      # opens a browser for Google OAuth
```

**The agent must not enter Google credentials or complete the consent screen itself** —
signing into an account and granting OAuth scopes are the human's actions. Run the command,
then tell the user to complete the browser sign-in. For a headless/server box, use a service
account instead:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json
# (key file supplied by the human; never fetch or generate credentials yourself)
```

### 3. Verify end-to-end

```bash
gws auth status                       # shows the signed-in account + scopes
gws gmail messages list --format json --max-results 1   # one real read
```

If `auth status` shows the account and the list call returns JSON, gws is working.

## Global flags worth knowing

| Flag | Use |
|---|---|
| `--format json\|table\|yaml\|csv` | json is the default and best for the agent to parse |
| `--dry-run` | validate locally without calling the API — use before any write |
| `--api-version <v>` | pin an API version if a default changes under you |

## Usage shape

```
gws <service> <resource> [sub-resource] <method> [flags]
```

e.g. `gws drive files list`, `gws sheets values get <id> <range>`, `gws calendar events list`.
A library of task-level gws skills (gmail triage, drive upload, doc writing, etc.) lives
separately; this skill only covers getting gws installed and authed.

## Safety

- **Never** enter the user's Google password or click through the OAuth consent — those are
  the human's actions (run `gws auth login`, then hand off).
- Prefer `--dry-run` before any send/delete/share. Sending mail, sharing files, and changing
  permissions are real-world side effects — confirm with the human first.
- On a client box, use the client's own Google account/service-account, not yours.

## Notes
- Update: `npm update -g @googleworkspace/cli` (or `npm i -g @googleworkspace/cli@latest`).
- Uninstall: `npm uninstall -g @googleworkspace/cli`.
- Repo / issues: https://github.com/googleworkspace/cli

## Last updated

2026-06-01 — initial author. Sourced from the live install on this machine
(`@googleworkspace/cli` 0.13.2, npm-global, official googleworkspace/cli repo).
