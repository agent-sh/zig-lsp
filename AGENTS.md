# zig-lsp

> Zig language server plugin for Claude Code, distributed via the agent-sh marketplace

## Marketplace

- agent-sh - this repo IS the marketplace; one plugin shipped today (`zig-lsp`)

## Plugins

- zig-lsp - registered via `lspServers` inline in `.claude-plugin/marketplace.json`, dispatching to user-installed `zls`

## Commands

None. This is a config-only plugin - no slash commands, no agents, no skills.

## Critical Rules

1. **Plain text output** - No emojis, no ASCII art. Use `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]` for status markers.
2. **No unnecessary files** - Don't create summary files, plan files, audit files, or temp docs.
3. **Task is not done until CI passes** - agnix workflow must be green before merge.
4. **Create PRs for non-trivial changes** - No direct pushes to main.
5. **Always run git hooks** - Never bypass pre-commit or pre-push hooks.
6. **Use single dash for em-dashes** - In prose, use ` - ` (single dash with spaces), never ` -- `.
7. **Mirror CLAUDE.md and AGENTS.md byte-identically** - When editing one, update the other in the same commit.
8. **Token efficiency** - Save tokens over decorations.

## Plugin shape - non-negotiables

1. **For third-party LSP plugins, ship `.lsp.json` at the plugin root** (`plugins/zig-lsp/.lsp.json`). This is the file the runtime loader reads from the user's cached install. We also keep `lspServers` inline in `marketplace.json` for marketplace-UI metadata and as a backstop. Both are required because official LSP plugins (typescript-lsp, rust-analyzer-lsp, etc.) load via a *separate* hardcoded path that doesn't apply to third parties. Empirically verified on Claude Code v2.1.119: with `.lsp.json` present in the cached plugin dir, `zig-lsp` registers as `Loaded 1 LSP server(s) from plugin: zig-lsp`; without it, the loader silently skips the entry even when `marketplace.json` has identical content.
2. **Don't include `restartOnCrash` or `maxRestarts` in `.lsp.json` or the `marketplace.json` `lspServers` block.** The plugins reference docs them, but the loader rejects them with `"restartOnCrash is not yet implemented. Remove this field from the configuration."` and the entire LSP server fails to initialize. `startupTimeout` works; the other lifecycle fields don't.
3. The `command` field is `"zls"` - a PATH-resolved binary name. Never embed an absolute path; users install the binary themselves.
4. `extensionToLanguage` maps both `.zig` and `.zon` to language ID `"zig"`. Don't drop `.zon` - the package manifest needs LSP support.
5. Bumping `zls` defaults: keep `.lsp.json` and the `lspServers` block in `marketplace.json` byte-equal. Update CHANGELOG and bump the plugin `version` in `plugin.json`.

## Validation

| Check | Command | Where |
|---|---|---|
| Plugin manifest | `claude plugin validate plugins/zig-lsp` | local |
| Marketplace manifest | `claude plugin validate .` | local |
| Agent config drift | `agnix --target claude-code` | local + CI |

CI runs `agent-sh/agnix@v0.20.1` against `.agnix.toml` on every push and PR.

## References

- [Claude Code plugin reference](https://code.claude.com/docs/en/plugins-reference#lsp-servers)
- [ZLS](https://github.com/zigtools/zls)
- [agent-knowledge/claude-code-lsp-plugin-contribution.md](agent-knowledge/claude-code-lsp-plugin-contribution.md) - research notes for contributors
- Part of the [agent-sh](https://github.com/agent-sh) org
