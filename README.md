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
│   └── marketplace.json     # Marketplace metadata (name: agent-sh)
├── plugins/
│   └── zig-lsp/
│       ├── .claude-plugin/
│       │   └── plugin.json  # Plugin manifest
│       ├── .lsp.json        # ZLS server config (the core file)
│       └── README.md        # User-facing setup + troubleshooting
├── agent-knowledge/         # Research notes for contributors
├── LICENSE
└── README.md
```

## Contributing

Issues and PRs welcome. The research notes in [`agent-knowledge/`](agent-knowledge/) document how Claude Code integrates LSP plugins, the `.lsp.json` schema, the contribution flow, and known pitfalls — start there.

## License

MIT.
