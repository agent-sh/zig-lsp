# agent-sh / zig-lsp

A Claude Code marketplace hosting the [`zig-lsp`](plugins/zig-lsp) plugin — Zig language support via [ZLS](https://github.com/zigtools/zls).

## Install

```
/plugin marketplace add agent-sh/zig-lsp
/plugin install zig-lsp@agent-sh
```

Open a `.zig` or `.zon` file. Claude Code reports diagnostics from `zls` automatically after every edit, and the `LSP` tool gives the model jump-to-definition, find-references, and hover.

You must have `zls` installed and on `PATH`. See [`plugins/zig-lsp/README.md`](plugins/zig-lsp/README.md) for setup, configuration, and troubleshooting.

## What this is

Claude Code's `LSP` tool dispatches to language-specific plugins. There are 11 official LSP plugins shipped by Anthropic; Zig isn't one of them yet. This plugin fills the gap with a thin manifest pointing at user-installed `zls`.

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json     # Marketplace metadata + ZLS server config
│                              (lspServers block — this is what Claude
│                              Code actually reads to register the LSP)
├── plugins/
│   └── zig-lsp/
│       ├── .claude-plugin/
│       │   └── plugin.json  # Plugin identity (name, version, license)
│       └── README.md        # User-facing setup + troubleshooting
├── agent-knowledge/         # Research notes for contributors
├── .agnix.toml              # agnix config (target: claude-code)
├── .github/workflows/       # agnix CI on push and PR
├── CHANGELOG.md
├── CONTRIBUTING.md
├── CLAUDE.md / AGENTS.md    # Project memory (byte-identical)
├── LICENSE
└── README.md
```

## Contributing

Issues and PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for local setup, validation, and the version-bump policy. The research notes in [`agent-knowledge/`](agent-knowledge/) document how Claude Code integrates LSP plugins, the `.lsp.json` schema, the contribution flow, and known pitfalls — useful background before opening a substantial PR.

## Changelog

See [CHANGELOG.md](CHANGELOG.md). Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## License

MIT — see [LICENSE](LICENSE).
