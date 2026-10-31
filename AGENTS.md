# zig-lsp

> Zig language server plugin for Claude Code, distributed via the agent-sh marketplace

## Marketplace

- agent-sh - this repo IS the marketplace; one plugin shipped today (`zig-lsp`)

## Plugins

- zig-lsp - registered via `plugins/zig-lsp/.lsp.json`, with `lspServers` in `.claude-plugin/marketplace.json` mirroring it; dispatches to user-installed `zls`

## Commands

None. This is a config-only plugin - no slash commands, no agents, no skills.

## Critical Rules

1. **Plain text output** - No emojis, no ASCII art. Use `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]` for status markers.
2. **No unnecessary files** - Don't create summary files, plan files, audit files, or temp docs.
3. **Task is not done until CI passes** - agnix workflow must be green before merge.
4. **Create PRs for non-trivial changes** - No direct pushes to main.
5. **Always run git hooks** - Never bypass pre-commit or pre-push hooks.
6. **Use single dash for em-dashes** - In prose, use ` - ` (single dash with spaces), never ` -- `.
7. **One instruction source** - Maintain repository guidance in `AGENTS.md`.
8. **Token efficiency** - Save tokens over decorations.

## Plugin shape - non-negotiables

1. **For third-party LSP plugins, ship `.lsp.json` at the plugin root** (`plugins/zig-lsp/.lsp.json`). This is the file the runtime loader reads from the user's cached install. We also keep `lspServers` inline in `marketplace.json` for marketplace-UI metadata and as a backstop. Both are required because official LSP plugins (typescript-lsp, rust-analyzer-lsp, etc.) load via a *separate* hardcoded path that doesn't apply to third parties. Empirically verified on Claude Code v2.1.119: with `.lsp.json` present in the cached plugin dir, `zig-lsp` registers as `Loaded 1 LSP server(s) from plugin: zig-lsp`; without it, the loader silently skips the entry even when `ma2. **Leave `restartOnCrash` and `maxRestarts` unset in `.lsp.json` and the `marketplace.json` `lspServers` block.** Current Claude Code (verified on 2.1.316) accepts both; we omit them on purpose so the defaults apply: a crashed server restarts, up to 3 times. Early Claude Code 2.1 loaders rejected them with `"restartOnCrash is not yet implemented. Remove this field from the configuration."` and the server failed to initialize.orks; the other lifecycle fields don't.
3. The `command` field is `"zls"` - a PATH-resolved binary name. Never embed an absolute path; users install the binary themselves.
4. `extensionToLanguage` maps both `.zig` and `.zon` to language ID `"zig"`. Don't drop `.zon` - the package manifest needs LSP support.
5. Bumping `zls` defaults: keep `.lsp.json` and the `lspServers` block in `marketplace.json` byte-equal. Update CHANGELOG and bump the plugin `version` in `plugin.json`.

## Validation

| Check | Command | Where |
|---|---|---|
| Plugin manifest | `claude plugin validate plugins/zig-lsp` | local |
| Marketplace manifest | `claude plugin validate .` | local |
| Agent config drift | `agnix --target claude-code` | local + CI |

CI runs `agent-sh/agnix@v0.56.5` against `.agnix.toml` on every push and PR.

## References

- [Claude Code plugin reference](https://code.claude.com/docs/en/plugins-reference#lsp-servers)
- [ZLS](https://github.com/zigtools/zls)
- [agent-knowledge/claude-code-lsp-plugin-contribution.md](agent-knowledge/claude-code-lsp-plugin-contribution.md) - research notes for contributors
- Part of the [agent-sh](https://github.com/agent-sh) org
