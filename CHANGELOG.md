# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Fixed
- **`.lsp.json` restored at plugin root.** Empirical testing on Claude Code v2.1.119 showed third-party LSP plugins need both: an `lspServers` block in `marketplace.json` (matches official-plugin shape) AND a `.lsp.json` file at the plugin root (this is what the runtime loader actually reads from the cached install directory). The previous round of changes deleted `.lsp.json` based on an incorrect inference from the official LSP plugins' minimal source layout — those plugins use a separate hardcoded loader path that doesn't apply to third parties. With `.lsp.json` shipped, `claude --debug` logs `Loaded 1 LSP server(s) from plugin: zig-lsp`. Without it, the loader silently skips the entry.

### Changed
- `agent-knowledge/claude-code-lsp-plugin-contribution.md` correction block expanded with the two-path picture (official hardcoded vs third-party `.lsp.json`) and the empirical evidence table.
- `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md` "non-negotiables" updated: `.lsp.json` and marketplace `lspServers` are both required, kept byte-equivalent on server config.
- Top-level `README.md` repo layout reflects `.lsp.json` back in the plugin source.

## [0.1.0] - 2026-04-26

### Added

- Initial `zig-lsp` plugin for Claude Code, dispatching to user-installed `zls`
- `agent-sh` marketplace shell hosting `zig-lsp` (and ready for future LSP plugins)
- `.lsp.json` mapping `.zig` and `.zon` to language ID `zig`, with `enable_build_on_save`, `restartOnCrash`, and a 30 s startup timeout
- `agent-knowledge/` research notes covering Claude Code's LSP plugin model, the `.lsp.json` schema, the contribution flow, and `zls` operator details (24 sources)
- `.agnix.toml` (target `claude-code`, `agent-knowledge/` excluded) and a CI workflow running `agent-sh/agnix@v0.20.1` on push and PR

[Unreleased]: https://github.com/agent-sh/zig-lsp/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/agent-sh/zig-lsp/releases/tag/v0.1.0
