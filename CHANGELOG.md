# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.1] - 2026-04-26

### Fixed
- **Removed `restartOnCrash` and `maxRestarts`** from `.lsp.json` and `marketplace.json`'s `lspServers` block. The Claude Code 2.1.x runtime documents these fields but rejects them at load time with `"restartOnCrash is not yet implemented. Remove this field from the configuration."`, causing the entire LSP server to fail initialization. With these fields present, `claude --debug` showed `Failed to initialize LSP server plugin:zig-lsp:zig` and `.zig` files got `No LSP server available`. Without them, ZLS spawns and `documentSymbol` / `hover` / post-edit diagnostics work end-to-end. `startupTimeout: 30000` is kept (loader accepts it).
- **`.lsp.json` restored at plugin root** (carried from Unreleased). Empirical testing on Claude Code v2.1.119 confirmed third-party LSP plugins need `.lsp.json` in the plugin source so it lands in the cached install dir; without it the loader silently skips the entry. Marketplace.json's `lspServers` field alone is empirically inert for non-`claude-plugins-official` marketplaces.

### Changed
- `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md` plug-shape rules now explicitly forbid `restartOnCrash` / `maxRestarts` until the runtime ships them; reflect the corrected two-file shape (`.lsp.json` + marketplace `lspServers`).
- `plugins/zig-lsp/README.md` configuration table no longer lists `restartOnCrash` / `maxRestarts`; gotchas section notes that a crashed ZLS stays dead until session restart.
- `agent-knowledge/claude-code-lsp-plugin-contribution.md` correction block adds the empirical row showing the loader rejects the unimplemented fields, and notes a third valid config location (`lspServers` inline in `.claude-plugin/plugin.json` — the path `michelsciortino/dart-lsp` uses; verified working on 2.1.119).
- Top-level `README.md` repo layout reflects `.lsp.json` back in the plugin source.

### Empirical verification
Tested end-to-end against `cockpit/src-tauri/sidecars/agent-probe-zig` (Zig 0.17, ~900 LOC `main.zig`). After `0.1.1` install:
- `Loaded 1 LSP server(s) from plugin: zig-lsp` in debug log
- ZLS spawned, `didOpen` succeeded, `publishDiagnostics` returned 0 issues
- `documentSymbol` returned 30+ top-level functions with line numbers
- `hover` on a struct identifier returned the full type definition

## [0.1.0] - 2026-04-26

### Added

- Initial `zig-lsp` plugin for Claude Code, dispatching to user-installed `zls`
- `agent-sh` marketplace shell hosting `zig-lsp` (and ready for future LSP plugins)
- `.lsp.json` mapping `.zig` and `.zon` to language ID `zig`, with `enable_build_on_save` and a 30 s startup timeout
- `agent-knowledge/` research notes covering Claude Code's LSP plugin model, the `.lsp.json` schema, the contribution flow, and `zls` operator details (24 sources)
- `.agnix.toml` (target `claude-code`, `agent-knowledge/` excluded) and a CI workflow running `agent-sh/agnix@v0.20.1` on push and PR

[Unreleased]: https://github.com/agent-sh/zig-lsp/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/agent-sh/zig-lsp/releases/tag/v0.1.1
[0.1.0]: https://github.com/agent-sh/zig-lsp/releases/tag/v0.1.0
