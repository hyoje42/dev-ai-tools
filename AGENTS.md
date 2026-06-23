# Agent Instructions

This repository (`dev-ai-tools`) is a meta-repo that manages the configuration of development AI tools (Claude Code, Codex) in one place. It is composed of two git submodules.

For the repo overview and setup, see [README.md](./README.md). This document holds the agent work rules plus an index that points to the right doc for each work location.

## Submodules

Each submodule follows the same pattern: **only `home/` is synced, and its structure is mirrored verbatim into `~/.<tool>/`.** Other root files (README, AGENTS, sync scripts, etc.) are for managing the repo and are not synced.

- **[claude-config/](./claude-config)** — `home/` syncs to `~/.claude/`.
- **[codex-config/](./codex-config)** — `home/` syncs to `~/.codex/`.

**Machine-specific settings** live outside the sync area, in each submodule's `local/` — the structure is committed as `*.example` and the real values are git-ignored. The application mechanism differs per tool (Claude = settings deep-merge, Codex = config.toml seed-if-absent). **Do not port one tool's mechanism onto the other.**

## Which doc to read for which work

Once the work target is decided, read the matching doc first and follow its guidance.

| Work target | Read first |
|---|---|
| Inside `claude-config/` | [claude-config/AGENTS.md](./claude-config/AGENTS.md) |
| Inside `codex-config/` | [codex-config/AGENTS.md](./codex-config/AGENTS.md) |
| Root level (submodule bumps, README, etc.) | This doc + [README.md](./README.md) |

> A submodule's AGENTS.md (= `CLAUDE.md`) is auto-loaded when you enter that subtree. Each submodule's detailed structure and usage lives in that repo's README.md.

## Work rules

- **Sync gate**: after editing `home/`, go only as far as checking the diff with `*-diff-with-home` and sharing it with the user. **Applying to `~/.<tool>/` (whether via `*-sync-to-home` or a manual copy) happens only when the user explicitly says so.** Do not sync on your own initiative.
- **Commit order**: commit in the submodule first → then commit the submodule pointer bump in the parent as a separate commit.
- The two submodules are versioned independently. Do not port a change from one to the other without conversion. When moving rules/skills between tools, always convert the tool-name and path differences (`.claude` ↔ `.codex`, Claude-only tool names, etc.) — see [codex-config/skill-authoring.md](./codex-config/skill-authoring.md) for the conversion rules.

## Remotes (mirroring to multiple remotes)

- **Keep the submodule URLs in `.gitmodules` as relative paths (`../claude-config.git`). Do not revert them to absolute URLs.** If you change a URL, run `git submodule sync` to apply it to the local `.git/config`. (See the README for why relative paths are needed.)
- **Push the submodules first, then the parent.** To keep multiple remotes (conventionally `origin`/`upstream` — see README) in sync, push to every remote so the mirrors don't diverge (the user may tell you to push only some remotes, so follow their instruction).

## Documentation principles

Split docs by **purpose** (no audience-based duplication).

- **README.md = repo description (canonical)** / **AGENTS.md (= `CLAUDE.md`) = work rules + index.** Do not duplicate the same content in both — point to the README or a dedicated doc for the overview and details.
- Pull longer topics (e.g., a skill authoring guide) into a **separate doc** and reference it from AGENTS in one line.
- AGENTS.md is auto-loaded, so keep it short (README is not auto-loaded).
- **Language**: agent-loaded instruction files (AGENTS.md — including Codex's `home/AGENTS.md` global instructions — Claude's `home/rules/`, and skills) are written in **English**; human-facing docs (README and notes/guides like `skill-authoring.md`, `settings-notes.md`, `docs/`) are written in **Korean**. Sync/utility scripts follow the same split — **user-facing output (`echo`/errors) in Korean**; code comments may stay in English.
