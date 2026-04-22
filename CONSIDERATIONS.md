# Considerations

## Target harness

This repo targets **Gemini CLI**. Do not install these workflow skills into a different coding-agent harness. The skill names are prefixed with `gemini-` to prevent cross-framework ambiguity.

Gemini-specific workflow skills are written for Gemini CLI Agent Skills and native skill activation/discovery.

## Runtime state

The old flat handoff/reflection layout is obsolete. Current workflow skills use the contract in `runtime-state.md`:

- `repo_hash = sha256(realpath(git rev-parse --show-toplevel))[:8]`
- `branch_slug = sanitized current branch`, or `detached-<short-sha>` when detached
- `run_id = <UTC YYYYMMDDTHHMMSSZ>-<short random suffix>`
- reflections: `$HOME/.gemini/skills/<skill>/reflections/<repo_hash>/<branch_slug>/<run_id>.md`
- handoffs: `$HOME/.gemini/skills/<skill>/handoffs/<repo_hash>/<branch_slug>/<run_id>.md`
- latest handoff pointer: `$HOME/.gemini/skills/<skill>/handoffs/<repo_hash>/<branch_slug>/latest.md`

Downstream skills must read `latest.md`, validate `from`, `repo`, `repo_root`, `branch`, `branch_slug`, `commit`, `run_id`, and `artifact`, and ignore mismatched handoffs unless the user explicitly asks to reuse cross-branch or cross-repo state.

## Architecture note for old sessions

If a running agent remembers unprefixed Claude-era names (`phase-roadmap-builder`, `plan-phase`, `execute-phase`, `plan-detailed`, `task-contextualizer`, `skill-improvement-planner`, `skill-editor`), redirect it to the prefixed skill in this repo. For example, use `gemini-plan-phase` instead of `plan-phase`.

## Skill groups

- `planning-chain/` contains the main roadmap -> plan -> execute loop.
- `meta/` contains the reflection aggregation and skill editing loop.
- `efficiency-kit/` contains passive utility skills. These are intentionally short and should not write runtime state.

## Installation target

Default install target: `$HOME/.gemini/skills`.

These framework-specific skills are intentionally not installed into `~/.agents/skills`. That directory should be reserved for future skills that are genuinely platform-neutral.

## Permission settings

Dotfiles manages Gemini CLI runtime access in `bootstrap.sh` by merging `tools.sandboxAllowedPaths` into `$HOME/.gemini/settings.json` for `$HOME/.gemini/skills` and the dotfiles `gemini-config/skills` symlink target. Gemini CLI does not currently provide a narrow path-specific edit auto-approval setting, so do not enable global `auto_edit` or YOLO mode just to reduce prompts for runtime artifacts.

## Style

Keep instructions directive-first. Avoid long narrative justification, war stories, or benchmark claims. If behavior differs between frameworks, do not hide that behind a generic skill; make an explicit framework-specific port.
