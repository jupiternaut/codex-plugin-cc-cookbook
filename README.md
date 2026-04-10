# codex-plugin-cc Cookbook

> A hands-on installation and usage guide for [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) on Windows — based on a real setup on Windows 11 with Claude Code.

---

## What is codex-plugin-cc?

`codex-plugin-cc` is an official OpenAI plugin for Claude Code that lets you run OpenAI Codex directly inside your Claude Code session. It gives you:

- **Code review** powered by Codex
- **Adversarial review** that challenges your design choices
- **Rescue tasks** — delegate bug fixes to Codex as a sub-agent
- **Background job management** — run long reviews without blocking

---

## Environment

This cookbook was written and tested on:

| Component | Version |
|-----------|---------|
| OS | Windows 11 Home |
| Shell | bash (Git Bash / MSYS2) |
| Node.js | v24.12.0 |
| npm | 11.6.2 |
| Codex CLI | codex-cli 0.118.0 |
| codex-plugin-cc | 1.0.3 |
| Claude Code | latest |

---

## Installation

> **Note:** The `/plugin marketplace add` slash command is not yet available in all Claude Code versions. The steps below install the plugin manually by writing directly to Claude Code's config directory.

### Step 1 — Install the Codex CLI

```bash
npm install -g @openai/codex
codex --version  # should print: codex-cli 0.118.0
```

### Step 2 — Clone the plugin repo

```bash
git clone --depth=1 https://github.com/openai/codex-plugin-cc.git /tmp/codex-plugin-cc
```

### Step 3 — Copy plugin files to Claude Code's marketplace directory

```bash
cp -r /tmp/codex-plugin-cc ~/.claude/plugins/marketplaces/openai-codex
```

On Windows the path resolves to:
```
C:\Users\<YourName>\.claude\plugins\marketplaces\openai-codex\
```

### Step 4 — Register the marketplace

Edit `~/.claude/plugins/known_marketplaces.json` and add the `openai-codex` entry:

```json
{
  "claude-plugins-official": {
    "source": { "source": "github", "repo": "anthropics/claude-plugins-official" },
    "installLocation": "C:\\Users\\<YourName>\\.claude\\plugins\\marketplaces\\claude-plugins-official",
    "lastUpdated": "2026-03-15T21:23:07.637Z"
  },
  "openai-codex": {
    "source": { "source": "github", "repo": "openai/codex-plugin-cc" },
    "installLocation": "C:\\Users\\<YourName>\\.claude\\plugins\\marketplaces\\openai-codex",
    "lastUpdated": "2026-04-10T00:00:00.000Z"
  }
}
```

### Step 5 — Install plugin skills

```bash
SKILL_SRC="$HOME/.claude/plugins/marketplaces/openai-codex/plugins/codex/skills"
SKILL_DST="$HOME/.claude/skills"

for skill in "$SKILL_SRC"/*/; do
  name=$(basename "$skill")
  cp -r "$skill" "$SKILL_DST/$name"
  echo "Installed skill: $name"
done
```

This installs three skills:
- `codex-cli-runtime`
- `codex-result-handling`
- `gpt-5-4-prompting`

### Step 6 — Verify setup

Run the setup check directly via Node:

```bash
CLAUDE_PLUGIN_ROOT="$HOME/.claude/plugins/marketplaces/openai-codex/plugins/codex" \
  node "$HOME/.claude/plugins/marketplaces/openai-codex/plugins/codex/scripts/codex-companion.mjs" setup --json
```

Expected output (when everything is working):

```json
{
  "ready": true,
  "node": { "available": true, "detail": "v24.12.0" },
  "npm":  { "available": true, "detail": "11.6.2" },
  "codex": { "available": true, "detail": "codex-cli 0.118.0; advanced runtime available" },
  "auth": {
    "available": true,
    "loggedIn": true,
    "detail": "ChatGPT login active for <your-email>",
    "source": "app-server",
    "authMethod": "chatgpt",
    "verified": true
  },
  "reviewGateEnabled": false
}
```

---

## Authentication

The plugin uses your existing Codex / ChatGPT login. If you have already signed into the [Codex desktop app](https://openai.com/codex) or the Codex CLI, authentication is automatic.

To log in manually via the CLI:

```bash
codex login
```

To verify your auth status:

```bash
codex auth status
```

---

## Commands

All commands are available as slash commands inside Claude Code.

### `/codex:setup`

Checks that Codex CLI is installed and authenticated. Optionally enables the review gate.

```
/codex:setup
/codex:setup --enable-review-gate
/codex:setup --disable-review-gate
```

**Review gate:** when enabled, Codex runs a fresh code review every time Claude Code is about to stop, acting as a quality gate before the session ends.

---

### `/codex:review`

Runs a read-only code review using Codex against your local git state.

```
/codex:review
/codex:review --wait
/codex:review --background
/codex:review --scope branch --base main
/codex:review --scope working-tree
```

**Arguments:**

| Flag | Effect |
|------|--------|
| `--wait` | Run in foreground, block until done |
| `--background` | Run as background job, returns immediately |
| `--base <ref>` | Compare against this git ref instead of working tree |
| `--scope auto\|working-tree\|branch` | What to review |

> For small changes (1-2 files) the plugin recommends `--wait`. For larger changesets it recommends `--background`.

---

### `/codex:adversarial-review`

Like `/codex:review` but more aggressive — challenges your design decisions and implementation choices.

```
/codex:adversarial-review
/codex:adversarial-review --wait focus on the auth module
/codex:adversarial-review --background --base main
```

**Additional argument:** free-text focus instructions after the flags tell Codex where to concentrate its critique.

---

### `/codex:rescue`

Delegates a fix or investigation to Codex as a sub-agent. Codex will make changes to your code.

```
/codex:rescue fix the null pointer exception in UserService
/codex:rescue --wait investigate why tests are flaky
/codex:rescue --background --effort high refactor the payment module
/codex:rescue --resume  # continue the last rescue job
/codex:rescue --fresh   # start a new rescue job
```

**Arguments:**

| Flag | Effect |
|------|--------|
| `--wait` | Foreground, stream output |
| `--background` | Background job |
| `--resume` | Continue the last job in this repo |
| `--fresh` | Force a new job even if one exists |
| `--model <model\|spark>` | Choose Codex model |
| `--effort <none\|minimal\|low\|medium\|high\|xhigh>` | Control how much Codex does |

---

### `/codex:status`

Shows active and recent Codex jobs for the current repository.

```
/codex:status
/codex:status --all
/codex:status <job-id>
/codex:status <job-id> --wait
```

---

### `/codex:result`

Retrieves the stored output of a completed Codex job.

```
/codex:result
/codex:result <job-id>
```

---

### `/codex:cancel`

Cancels an active background job.

```
/codex:cancel
/codex:cancel <job-id>
```

---

## Typical Workflows

### Quick review before committing

```
/codex:review --wait
```

### Background review while you keep working

```
/codex:review --background
# ... keep working ...
/codex:status
/codex:result
```

### Branch diff review before opening a PR

```
/codex:review --scope branch --base main --background
```

### Delegate a hard bug fix to Codex

```
/codex:rescue --background --effort high the login flow breaks when the token is expired
/codex:status   # check if done
/codex:result   # read what Codex did
```

### Adversarial review of a PR branch

```
/codex:adversarial-review --background --base main focus on security and edge cases
```

---

## File Locations (Windows)

| Path | Purpose |
|------|---------|
| `%USERPROFILE%\.claude\plugins\known_marketplaces.json` | Marketplace registry |
| `%USERPROFILE%\.claude\plugins\marketplaces\openai-codex\` | Plugin source files |
| `%USERPROFILE%\.claude\skills\codex-cli-runtime\` | Skill: CLI runtime |
| `%USERPROFILE%\.claude\skills\codex-result-handling\` | Skill: result handling |
| `%USERPROFILE%\.claude\skills\gpt-5-4-prompting\` | Skill: prompting style |
| `%USERPROFILE%\.claude\plugins\marketplaces\openai-codex\plugins\codex\commands\` | Slash command definitions |
| `%USERPROFILE%\.claude\plugins\marketplaces\openai-codex\plugins\codex\hooks\hooks.json` | Session lifecycle hooks |
| `%USERPROFILE%\.claude\plugins\marketplaces\openai-codex\plugins\codex\scripts\` | Node.js runtime scripts |

---

## Troubleshooting

**`Unknown skill: codex:setup`**

The slash commands (`/codex:*`) are loaded from the marketplace directory, not from the skills directory. If you see this error, Claude Code may not yet have loaded the marketplace. Restart Claude Code after completing the installation steps.

**`auth.available: false`**

Run `codex login` in your terminal to authenticate through the browser, then re-run the setup check.

**Hooks not firing**

The plugin registers `SessionStart`, `SessionEnd`, and `Stop` hooks via `hooks.json`. These require `${CLAUDE_PLUGIN_ROOT}` to be resolved correctly. If hooks don't fire, check that the plugin's `installLocation` in `known_marketplaces.json` uses the correct absolute Windows path.

**Windows path separators**

Use double backslashes (`\\`) in `known_marketplaces.json` for `installLocation`. Bash paths with forward slashes work fine in terminal commands.

---

## Links

- [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) — official plugin repo
- [OpenAI Codex](https://openai.com/codex) — Codex product page
- [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) — Claude Code documentation
