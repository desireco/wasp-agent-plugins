# Changelog

All notable changes to the Wasp Claude Code plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.4.0] - 2026-06-23

### Added

- Documented Wasp's automatic Entity-based Query cache invalidation in `general-wasp-knowledge.md`, with guidance on when (not) to call `invalidateQueries` manually and where to import `useQueryClient` from (`@tanstack/react-query`, not `wasp/client/operations`).
- Added "Adding a new operation (the type-bootstrap loop)" section explaining the declare → recompile (`wasp start` watches, else `wasp compile`) → type-resolves sequence that causes expected `Cannot find name 'GetX'` errors.
- Added a client-side import cheat-sheet covering `wasp/client/operations`, `wasp/server/operations`, `wasp/entities`, `wasp/server/auth`, and `@tanstack/react-query`.
- Added "Migrations in non-interactive (agent) shells" covering the `migrate dev` TTY abort: don't run raw `npx prisma`, ask the user to run `wasp db migrate-dev` in their terminal.
- Added "Seeding the Database" section covering `wasp db seed` run/prompt behavior, seed idempotency, and the database-wide vs. per-user-default distinction.
- Added "Creating a verified user for E2E / integration tests" using `sanitizeAndSerializeProviderData` from `wasp/server/auth` (the documented helper, including `emailVerificationSentAt` / `passwordResetSentAt`).
- Expanded the Common Mistakes table with rows for generated-type errors, non-interactive migration failures, `useQueryClient` import source, and missing UI refresh after mutations.

### Changed

- Softened the Operations guidance from "DO NOT use `useAction`" to "use it when you need optimistic updates" (its documented purpose as Wasp's only native manual cache-invalidation mechanism).
- `start-dev-server` no longer hard-codes `localhost:3000`/`3001`; it now reads `WASP_WEB_CLIENT_URL` / `PORT` from `.env.server` (falling back to the defaults) so projects with custom ports validate correctly.

## [1.3.0] - 2026-03-24

### Added

- Multi-agent support: skills now work with Codex, Gemini, Copilot, OpenCode, and other agents via AGENTS.md
- Installation path for non-Claude agents via `npx skills add wasp-lang/claude-plugins`
- `wasp-plugin-init` skill now asks whether the user is on Claude Code (CLAUDE.md) or another agent (AGENTS.md) and follows the appropriate flow
- Chrome DevTools MCP install instructions for non-bundled setups

### Changed

- Agent-agnostic language throughout all skills ("Claude" → "the agent")
- Moved `general-wasp-knowledge.md` into the `wasp-plugin-init` skill directory
- Versioned init marker files (`.wasp-plugin-initialized-v{VERSION}`) for automatic re-init on plugin upgrades
- Simplified init detection: replaced CLAUDE.md content scanning with marker file approach
- Moved opt-out marker from `.claude/knowledge/.wasp-init-opt-out` to `.claude/wasp/.wasp-plugin-opt-out` (legacy path still supported)
- Chrome DevTools MCP is no longer bundled in the plugin manifest — users install it separately

## [1.2.0] - 2026-02-04

### Changed

- Converted commands to skills: `wasp:init` → `/wasp-plugin-init`, `wasp:help` → `/wasp-plugin-help`
- Renamed `expert-advice` command to skill
- Renamed `plugin-help` skill to `wasp-plugin-help`

### Removed

- Removed `commands/` directory (functionality moved to skills)

## [1.1.1] - 2026-01-27

### Added

- Added Chrome DevTools MCP server to plugin manifest

## [1.1.0] - 2026-01-21

### Changed

- Simplified recommended permissions to use wildcards (`Bash(wasp:*)`, `Skill(wasp:*)`)
- Consolidated database migration guidance into main skill files
- Improved documentation access section in plugin-help skill
- Updated styling skill with clearer instructions

### Removed

- Removed standalone "Verify Setup" feature (functionality covered by start-dev-server skill)
- Removed `running-db-migrations.md` (consolidated into start-dev-server SKILL.md)
- Removed `verify-setup.md` (no longer needed as separate feature)

### Added

- Added `.claude-plugin/plugin.json` manifest file

## [1.0.0] - 2025-12-11

### Added

- Initial release
- Wasp documentation integration with version detection
- Skills: add-feature, plugin-help, start-dev-server
- Commands: init, expert-advice
- Session start hook for initialization check
- Chrome DevTools MCP integration
