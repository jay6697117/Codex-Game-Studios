# Findings

## Repository Shape

- The repository currently appears to be a documentation-heavy workspace.
- Top-level files include `README.md`, `UPGRADING.md`, `CLAUDE.md`, `.github`, `docs`, and `production`.
- No application source trees or build manifests were found in the initial scan.

## Early Hypothesis

- This project is likely intended as an operational knowledge base and workflow system for AI-assisted game studio work, not as a single runnable application.

## Core Positioning

- `README.md` frames the repository as a template that turns one Claude Code session into a multi-agent game studio.
- The primary deliverables are agent definitions, skills, hooks, rules, templates, and engine-reference documents rather than executable product code.
- The repository expects downstream users to copy or clone it into their own game repo and then fill in engine, language, source tree, assets, design docs, and production artifacts.

## Workflow Model

- The intended collaboration model is explicitly non-autonomous: ask questions, present options, let the user decide, then draft, then request approval before writing.
- `CLAUDE.md` is deliberately slim and delegates detail into imported docs, which suggests a layered configuration strategy.
- `docs/WORKFLOW-GUIDE.md` defines an end-to-end process from concept, pre-production, prototyping, production, QA, polish, release, and live ops.

## Upgrade and Evolution Signals

- `UPGRADING.md` shows the template is versioned and actively evolving.
- Recent documented changes focus on workflow refinements, naming cleanups, hook fixes, and better stage/status tracking.

## Structural Consistency

- The headline inventory in `README.md` matches the repository contents:
  - 48 agent files
  - 37 skill files
  - 8 hook scripts
  - 11 rules files
  - 29 templates
- This is a strong signal that the repository is curated as a productized template rather than an ad hoc note dump.

## Configuration Architecture

- `.claude/settings.json` acts as the operational control plane:
  - status line integration
  - allow/deny permission patterns
  - lifecycle hooks for session start, tool usage, compaction, stop, and subagent logging
- `CLAUDE.md` stays intentionally compact and imports additional docs, which reduces duplication and makes the top-level contract easier to audit.

## State and Recovery Design

- `.claude/docs/context-management.md` treats filesystem state as primary memory and conversation state as disposable.
- The recommended `production/session-state/active.md` checkpoint pattern is a serious attempt to make long-running agent work resilient to context loss.

## Agent and Skill Design Direction

- Representative agent docs, such as `technical-director`, are written as decision-making role prompts, not implementation prompts.
- Representative skills, such as `start`, are workflow routers that diagnose project maturity and hand the user to the correct next process.
- `detect-gaps.sh` encodes repository heuristics for identifying missing concept docs, ADRs, production planning, and prototype documentation.

## Template Maturity

- The repository is still in a clean starter-template state:
  - `.claude/docs/technical-preferences.md` is entirely placeholder-driven.
  - No `src/`, `assets/`, `design/`, `tests/`, `tools/`, or `prototypes/` directories exist yet.
  - `production/` currently only contains `production/session-state/.gitkeep`.
- This means the current repository should be analyzed as infrastructure for future game repos, not as a partially implemented game.

## Engine Reference Strategy

- `docs/engine-reference/` contains 46 markdown files covering Godot, Unity, and Unreal.
- The structure is coherent across engines:
  - version pin
  - breaking changes
  - deprecated APIs
  - current best practices
  - 8 subsystem modules per engine
  - plugin docs where relevant
- The intent is solid: constrain model hallucination by pinning post-cutoff engine knowledge locally.

## Engine Reference Quality Gaps

- The repository's own maintenance rule says module files should stay under ~150 lines for context efficiency.
- Actual state diverges materially:
  - 16 of 24 module files exceed 150 lines
  - 7 of 7 plugin files exceed 150 lines
- Some representative files are far above the stated target:
  - `docs/engine-reference/unreal/modules/networking.md` at 409 lines
  - `docs/engine-reference/unity/modules/ui.md` at 377 lines
  - `docs/engine-reference/unreal/plugins/gameplay-ability-system.md` at 386 lines
- Conclusion: the knowledge-base strategy is good, but the current document granularity is drifting beyond the intended context budget.

## Concrete Implementation Issue

- `.claude/statusline.sh` appears to parse the engine line using `^\*\*Engine\*\*:` while `.claude/docs/technical-preferences.md` stores it as `- **Engine**: ...`.
- This likely causes `engine_configured` auto-detection to fail unless stage is pinned elsewhere.
- Resulting risk: status line stage detection can lag behind actual project configuration, reducing trust in automation feedback.

## Documentation Surface

- `docs/examples/` is a strong onboarding asset because it demonstrates intended collaboration style with concrete transcripts rather than only abstract principles.
- `.github/` currently provides issue and PR templates, but there is no visible CI workflow in the repository.

## Small Content Gap

- `docs/engine-reference/README.md` mentions `/refresh-docs` as a possible maintenance path, but no such skill or script is present in the repository snapshot.
- This is minor, but it creates a discoverability gap for maintaining the engine-reference corpus.

## Codex Adaptation Research

- Official Codex customization primitives differ from Claude Code:
  - project guidance via `AGENTS.md`
  - repo or global skills
  - project-scoped `.codex/config.toml`
  - MCP configuration
  - optional multi-agent role definitions in config
- Codex officially supports repo skills as `.agents/skills`, but `skills.config` in `.codex/config.toml` can also register arbitrary skill paths. This makes a `.codex/skills/` layout feasible without touching the original `.claude/skills/`.
- Codex discovers project guidance from root or nested `AGENTS.md` files by walking from project root to current working directory. There is no documented project-scoped `.codex/AGENTS.md` discovery path.
- Because the user requires all adaptation artifacts to live under `.codex/`, the most practical strategy is:
  - keep the original Claude files untouched
  - use `.codex/config.toml` as the Codex entrypoint
  - register repo-local skills and agent-role configs from `.codex/`
  - move durable project policy into `.codex/` documents plus concise config-driven guidance
- Multi-agent support in Codex is configured via the `[agents]` section in `.codex/config.toml`, with role descriptions and optional per-role config files.
- I did not find a documented Codex equivalent to Claude Code's project hook system (`SessionStart`, `PreToolUse`, `PostToolUse`, `PreCompact`, `Stop`, `SubagentStart`).
- I also did not find a documented custom-script statusline equivalent. Codex exposes built-in footer status-line items configurable through `/statusline` and `tui.status_line`.
- Inference: Claude hooks and custom statusline scripts should not be ported 1:1. They should be re-expressed as:
  - explicit skills
  - reusable validation scripts invoked by those skills
  - optional automations or MCP integrations where appropriate
