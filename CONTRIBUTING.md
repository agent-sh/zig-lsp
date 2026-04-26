# Contributing to zig-lsp

Thanks for considering a contribution. This is a thin Claude Code plugin - the surface is small, but a few conventions matter.

## Local setup

Clone, then sideload the plugin into a Claude Code session pointed at any Zig project:

```sh
git clone https://github.com/agent-sh/zig-lsp
cd zig-lsp

# In a separate Zig project:
claude --plugin-dir /absolute/path/to/zig-lsp/plugins/zig-lsp --debug
```

In the debug output, look for `Total LSP servers loaded: 1`. If it's `0`, the manifest isn't being picked up - usually because `.lsp.json` ended up inside `.claude-plugin/` instead of the plugin root.

You'll need [`zls`](https://github.com/zigtools/zls) on `PATH`. ZVM is recommended (`zvm i master --zls`) since it keeps `zls` and `zig` versions in lockstep.

## Validating before you push

Two checks must pass locally and in CI:

```sh
claude plugin validate plugins/zig-lsp     # plugin manifest
claude plugin validate .                   # marketplace manifest
agnix --target claude-code                 # config drift across the repo
```

`agnix` runs in CI via `agent-sh/agnix@v0.20.1` against `.agnix.toml`. The CI gate is `--fail-on-error`, so warnings don't fail builds but errors do.

## What changes need a version bump

`plugins/zig-lsp/.claude-plugin/plugin.json` `version` follows semver:

| Change | Bump |
|---|---|
| Bug fix in `lspServers` defaults that doesn't change behaviour for correct setups | patch (`0.1.0` -> `0.1.1`) |
| New `initializationOptions`, new mapped extension, defaults that change observable behaviour | minor (`0.1.0` -> `0.2.0`) |
| Schema break (renaming the server key, removing options) | major (`0.x` -> `1.0`) |

Update `CHANGELOG.md` in the same PR. Keep entries under the `## [Unreleased]` heading until release; the release commit moves them under a dated heading.

## Plugin shape - non-negotiables

1. **Ship `.lsp.json` at the plugin root** (`plugins/zig-lsp/.lsp.json`) AND keep an `lspServers` block inline in `.claude-plugin/marketplace.json`. Both are required for a third-party LSP plugin: the runtime loader reads `.lsp.json` from the user's cached install (this is what makes the LSP register on Claude Code v2.1.119); the marketplace `lspServers` field is metadata for the marketplace UI and matches what official plugins ship. Official LSP plugins (typescript-lsp, rust-analyzer-lsp) load via a separate hardcoded code path inside the Claude Code binary and don't ship `.lsp.json` themselves; that path doesn't apply to third parties.
2. The `command` field is `"zls"` - a PATH-resolved binary name. Never embed an absolute path; users install the binary themselves.
3. `extensionToLanguage` maps both `.zig` and `.zon` to language ID `"zig"`. Don't drop `.zon` - `build.zig.zon` needs LSP support.
4. Keep `.lsp.json` and the `lspServers` block in `marketplace.json` byte-equal on the server config. If they drift, behavior depends on which file the loader picks up first.
5. `CLAUDE.md` and `AGENTS.md` are mirrored byte-identically. When editing one, update the other in the same commit.

## Style

- Plain text in user-facing output. No emojis, no ASCII art. Use `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]` for status markers.
- Single dash for em-dashes (` - `), never ` -- `.
- Don't add files the change doesn't need - no plan docs, no summary docs, no scratch notes in the repo.

## Pull request flow

1. Fork, branch off `main`.
2. Make your change, run validation locally.
3. Update `CHANGELOG.md` under `## [Unreleased]`.
4. Open a PR. The agnix workflow runs automatically; it must be green to merge.
5. No direct pushes to `main` for non-trivial changes.

## Reporting issues

Open an issue at <https://github.com/agent-sh/zig-lsp/issues>. For LSP misbehaviour, include:

- `zls --version` output
- `zig version` output (mismatch is the most common cause of crashes)
- The output of `claude --debug` showing the `Total LSP servers loaded` line
- A minimal `.zig` reproducer if relevant

## License

By contributing you agree your work will be released under the [MIT License](LICENSE).
