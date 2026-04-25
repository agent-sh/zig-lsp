# zig-lsp

> Zig language server plugin for Claude Code, distributed via the agent-sh marketplace

## Marketplace

- agent-sh - this repo IS the marketplace; one plugin shipped today (`zig-lsp`)

## Plugins

- zig-lsp - thin `.lsp.json` manifest pointing at user-installed `zls`

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

1. `.lsp.json` lives at the **plugin root** (`plugins/zig-lsp/.lsp.json`), not inside `.claude-plugin/`. Putting it inside `.claude-plugin/` silently breaks loading.
2. The `command` field is `"zls"` - a PATH-resolved binary name. Never embed an absolute path; users install the binary themselves.
3. `extensionToLanguage` maps both `.zig` and `.zon` to language ID `"zig"`. Don't drop `.zon` - the package manifest needs LSP support.
4. Bumping `zls` defaults in `.lsp.json` is a user-facing change - update CHANGELOG and bump the plugin `version` in `plugin.json`.

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
