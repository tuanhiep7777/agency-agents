# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

The Agency is a content library, not an application: ~450+ specialized AI agent personas defined as markdown files, organized into divisions (`engineering/`, `design/`, `marketing/`, `sales/`, `security/`, etc.). A bash toolchain converts and installs these agent files into 15+ different AI coding tools (Claude Code, Cursor, Codex, Gemini CLI, Copilot, Qwen, OpenCode, Aider, Windsurf, OpenClaw, Osaurus, Antigravity, Kimi, Hermes, Mistral Vibe).

There is no build step, compiler, or test suite in the usual sense — "correctness" here means valid agent markdown and consistent JSON/script configuration, checked by the scripts below.

## Commands

```bash
# Render all agent .md files into tool-specific formats under integrations/<tool>/
./scripts/convert.sh                  # all tools
./scripts/convert.sh --tool cursor    # one tool
./scripts/convert.sh --parallel --jobs N

# Install converted agents into the user's actual tool config dirs
./scripts/install.sh                                   # interactive wizard (TTY)
./scripts/install.sh --tool claude-code --division engineering,security
./scripts/install.sh --tool cursor --agent frontend-developer,ui-designer
./scripts/install.sh --list teams                       # list divisions + agent counts
./scripts/install.sh --dry-run

# Validate agent markdown (frontmatter, recommended sections, LF line endings)
./scripts/lint-agents.sh                # all agents
./scripts/lint-agents.sh path/to/agent.md

# Flag near-duplicate / re-skinned agents (requires python3)
./scripts/check-agent-originality.sh path/to/new-agent.md

# Consistency checks for the two JSON sources of truth (no deps beyond bash+coreutils)
./scripts/check-divisions.sh
./scripts/check-tools.sh
```

CI (`.github/workflows/lint-agents.yml`, `check-divisions.yml`, `check-tools.yml`) runs exactly these scripts against changed files — there's nothing else to run before opening a PR.

## Architecture

- **`divisions.json`** — source of truth for the division set (dir name, label, icon, color). Must agree with the directories on disk, `AGENT_DIRS` in `scripts/convert.sh` and `scripts/lint-agents.sh`, and the path filters in `.github/workflows/lint-agents.yml`. Enforced by `check-divisions.sh`. `strategy/` (NEXUS playbooks, no agent frontmatter) and `examples/` are *not* divisions.

- **`tools.json`** — source of truth for the supported tool set: detect dirs, dest paths, render `format`, and `installKind` (`per-agent` = one file per agent, `roster` = one combined file, `plugin` = built CLI-only artifact). Must agree with `ALL_TOOLS` in `scripts/install.sh` and the converter set in `scripts/convert.sh`. Enforced by `check-tools.sh`.

- **`scripts/lib.sh`** — shared, dependency-free bash helpers sourced by both `convert.sh` and `install.sh`: frontmatter parsing (`get_field`, `get_body`), slug derivation (`slugify`, `agent_slug`), and the TUI primitives behind `install.sh`'s interactive wizard.

- **Agent file format** — YAML frontmatter (`name`, `description`, `color` required; `emoji`, `vibe`, optional `services` list) followed by a markdown body. `convert.sh` classifies each `##` section header into one of two semantic groups to render tool-specific splits (e.g. OpenClaw's `SOUL.md` vs `AGENTS.md`):
  - **Persona** — Identity & Memory, Communication Style, Critical Rules
  - **Operations** — Core Mission, Technical Deliverables, Workflow Process, Success Metrics, Advanced Capabilities

- **`integrations/`** — generated, gitignored per-tool output written by `convert.sh`. Never hand-edit or commit files here; regenerate instead.

## Contribution constraints

- A single new or improved agent `.md` file is always a welcome, mergeable PR on its own.
- New scripts, new directories, new tool integrations, or bulk reformatting across many existing agents should start as a GitHub Discussion first — not land as a surprise PR.
- New agents must pass `check-agent-originality.sh` — it fails PRs that are find-replace re-skins of an existing agent (e.g. swapping a country/platform name).
- Adding a division or a tool touches several files that must stay in lockstep (see Architecture above); run `check-divisions.sh` / `check-tools.sh` after and fix whatever they flag.
