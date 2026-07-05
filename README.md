# codex-plugin-cc Cookbook

This repository is a README-only cookbook for setting up and using `openai/codex-plugin-cc` with Claude Code. It is documentation, not an application or library.

## Current Status

- Status: documentation repository.
- Branch: `master`.
- Source files: none.
- Build output: none.
- Runtime: none in this repository.

The commands in this README are instructions to run in a separate Claude Code and Codex CLI environment. Cloning this repository does not install or run the plugin.

## What This Cookbook Covers

- Installing the Codex CLI.
- Cloning the upstream `openai/codex-plugin-cc` repository.
- Registering the plugin with Claude Code when marketplace installation is not available.
- Installing plugin skills into the local Claude Code skills directory.
- Running the plugin setup check.
- Understanding the plugin slash commands at a high level.

## Dependencies

To follow the cookbook on your own machine, you need:

- Claude Code.
- Node.js and npm.
- Git.
- Codex CLI.
- An authenticated Codex or ChatGPT session that the Codex CLI can use.
- A shell appropriate for your OS. The original notes were written from a Windows 11 setup using Git Bash/MSYS2-style commands.

This repository itself has no `package.json`, no source tree, and no dependency installation step.

## Use From a Fresh Clone

```bash
git clone https://github.com/jupiternaut/codex-plugin-cc-cookbook.git
cd codex-plugin-cc-cookbook
```

Then read `README.md` and run only the commands that match your local operating system, Claude Code version, and Codex authentication setup.

## Build, Run, and Test

There is nothing to build or run in this repository.

To validate the cookbook instructions, run the documented setup check in the target Claude Code plugin environment after installing the real plugin:

```bash
CLAUDE_PLUGIN_ROOT="$HOME/.claude/plugins/marketplaces/openai-codex/plugins/codex" \
  node "$HOME/.claude/plugins/marketplaces/openai-codex/plugins/codex/scripts/codex-companion.mjs" setup --json
```

A successful validation should report that Node, npm, Codex CLI, and authentication are available. Exact fields may change with upstream plugin releases.

## Documentation Boundaries

- This repository does not vendor `openai/codex-plugin-cc`.
- This repository does not contain a Claude Code plugin manifest.
- This repository does not create, store, or manage API keys.
- Commands that modify `~/.claude` or install global npm packages affect your local machine, not this repository.
- Upstream Claude Code, Codex CLI, and plugin behavior can change; treat version-specific instructions as a snapshot.

## Screenshots

No screenshot is currently checked in. Suggested placeholder path for future documentation:

```text
docs/screenshots/codex-plugin-cc-setup-check.png
```

## Issues and Contributions

Use GitHub Issues for stale setup steps, missing OS-specific notes, or unclear command output. Pull requests should keep this repository documentation-only unless the project owner explicitly changes its scope.
