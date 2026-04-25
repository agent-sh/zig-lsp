# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-04-26

### Added

- Initial `zig-lsp` plugin for Claude Code, dispatching to user-installed `zls`
- `agent-sh` marketplace shell hosting `zig-lsp` (and ready for future LSP plugins)
- `.lsp.json` mapping `.zig` and `.zon` to language ID `zig`, with `enable_build_on_save`, `restartOnCrash`, and a 30 s startup timeout
- `agent-knowledge/` research notes covering Claude Code's LSP plugin model, the `.lsp.json` schema, the contribution flow, and `zls` operator details (24 sources)
- `.agnix.toml` (target `claude-code`, `agent-knowledge/` excluded) and a CI workflow running `agent-sh/agnix@v0.20.1` on push and PR

[Unreleased]: https://github.com/agent-sh/zig-lsp/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/agent-sh/zig-lsp/releases/tag/v0.1.0
