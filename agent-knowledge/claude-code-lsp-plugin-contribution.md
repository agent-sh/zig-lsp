# Learning Guide: Contributing a Zig / ZLS LSP Plugin to Claude Code

**Generated**: 2026-04-26
**Sources**: 24 resources analyzed
**Depth**: medium

> **Key facts for retrieval**: The plugin is 3 files (`zig-lsp/.lsp.json`, `zig-lsp/.claude-plugin/plugin.json`, `zig-lsp/README.md`). Binary name is `zls`. Extensions: `.zig` and `.zon` → language `"zig"`. Official marketplace submission is form-gated (not PR-based). All 11 official LSP plugins had a bug (missing `.lsp.json`) fixed in PR #378 — always ship `.lsp.json` as a physical file. Test locally with `claude --plugin-dir ./zig-lsp`.

---

## TL;DR — What to do to ship a Zig plugin (5 concrete actions)

1. **Create the plugin directory** at `zig-lsp/` with exactly two required files: `.lsp.json` (LSP server config) and `.claude-plugin/plugin.json` (plugin manifest). Add a `README.md`. That is the complete plugin.
2. **Write `.lsp.json`** mapping `command: "zls"`, extensions `.zig` and `.zon` to language `"zig"`, with `startupTimeout: 30000`, `restartOnCrash: true`, `maxRestarts: 3`, and `enable_build_on_save` in `initializationOptions`.
3. **Test locally** before submitting: `claude --plugin-dir ./zig-lsp` — open a `.zig` file and confirm diagnostics appear after a save. Check `~/.claude/debug/latest` for `Total LSP servers loaded: 1`.
4. **Submit to the official marketplace** via the in-app form at `claude.ai/settings/plugins/submit` or `platform.claude.com/plugins/submit`. The target repo is `github.com/anthropics/claude-plugins-official`; plugins go under `plugins/zig-lsp/`. There is no PR-based open contribution path currently — submission is form-gated.
5. **Ship immediately as a community plugin** by pushing your repo to GitHub and advertising the install command: `/plugin marketplace add <your-github-username>/zig-lsp && /plugin install zig-lsp@<your-marketplace-name>`. Community LSP plugins that ship their own marketplace are fully functional today.

---

## Prerequisites

**What you already know** (covered in your reference doc at `../tools/agent-knowledge/lsp-tool-design-across-harnesses.md`):
- The `LSP` tool dispatch model — one tool, plugin-per-language manifest
- The harness/plugin boundary: harness owns lifecycle, plugin owns config
- Manifest field names and the `gopls` example
- The `textDocument/publishDiagnostics` hook behavior (automatic, not wired by contributor)
- How Claude Code compares to OpenCode, Codex, and Cursor on LSP integration

**What this guide adds — the plugin author's view**:
- Exact directory layout of a shipped Anthropic LSP plugin (from PR #378 source)
- The current `.lsp.json` schema with types and defaults (authoritative as of 2026-04-26)
- The `marketplace.json` schema for self-hosting
- The `zls` operator's view: binary name, install paths, extension list, `initializationOptions` keys
- A complete worked manifest for `zls`
- Local testing commands including LSP-specific debug flags
- Open bugs and gotchas as of Q1 2026

---

## How a Claude Code LSP Plugin Is Structured

### Directory layout (authoritative)

The canonical structure for an LSP plugin, derived from PR #378 which added `.lsp.json` files to all 11 official plugins:

```
zig-lsp/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest — name, version, author, description
├── .lsp.json             # LSP server configuration (the core file)
└── README.md             # Required by the official marketplace; install + binary instructions
```

LSP plugins need no `skills/`, `hooks/`, or `agents/` directories.

**Critical layout rule**: only `plugin.json` goes inside `.claude-plugin/`. The `.lsp.json` is at the **plugin root**, not inside `.claude-plugin/`. This is the most common structural mistake.

### File-by-file walkthrough — gopls-lsp as the reference

`plugins/gopls-lsp/.lsp.json` (from PR #378):
```json
{
  "gopls": {
    "command": "gopls",
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

`plugins/gopls-lsp/README.md`:
```markdown
# gopls-lsp

Go language server for Claude Code, providing code intelligence, refactoring, and analysis.

## Supported Extensions
`.go`

## Installation
Install gopls using the Go toolchain:
```
go install golang.org/x/tools/gopls@latest
```
Make sure $GOPATH/bin (or $HOME/go/bin) is in your PATH.

## More Information
- [gopls Documentation](https://pkg.go.dev/golang.org/x/tools/gopls)
```

The `.claude-plugin/plugin.json` for official plugins is minimal — name, version, author. The community plugin `cc-zig-lsp` (4rgon4ut) uses this pattern:
```json
{
  "name": "zig-lsp",
  "description": "Zig language support via ZLS",
  "version": "0.1.0",
  "author": {
    "name": "Your Name"
  },
  "repository": "https://github.com/yourname/zig-lsp",
  "keywords": ["zig", "lsp", "zls", "language-server"],
  "lspServers": "./.lsp.json"
}
```

Note: `lspServers` in `plugin.json` can point to the file (`./.lsp.json`) or inline the object directly. The official marketplace convention is a separate `.lsp.json` at root.

### Startup/timeout patterns across existing plugins

| Plugin | `startupTimeout` | `args` | Notes |
|--------|-----------------|--------|-------|
| `gopls-lsp` | omitted | omitted | Fast native binary |
| `jdtls-lsp` | `120000` (2 min) | omitted | JVM startup is slow |
| `kotlin-lsp` | `120000` (2 min) | `["--stdio"]` | JVM startup is slow |
| `pyright-lsp` | omitted | `["--stdio"]` | Node-based, fast |
| `typescript-lsp` | omitted | `["--stdio"]` | Node-based, fast |
| `rust-analyzer-lsp` | omitted | omitted | Rust binary, reasonable startup |
| `clangd-lsp` | omitted | `["--background-index"]` | Background indexing flag |

`zls` is a native Zig binary with fast cold startup (comparable to `gopls`). Use `startupTimeout: 30000` as a conservative safe default.

---

## The `.lsp.json` Manifest Schema

Source: [Plugins reference — LSP servers](https://code.claude.com/docs/en/plugins-reference#lsp-servers) (verified 2026-04-26)

### Required fields

| Field | Type | Description |
|-------|------|-------------|
| `command` | `string` | The LSP binary name. Must be resolvable via `PATH`. Do not embed a path here — users install the binary themselves. |
| `extensionToLanguage` | `object` | Maps file extension strings (including the `.`) to LSP language identifier strings. Both key and value are strings. |

The top-level key of the config object (e.g. `"zig"`, `"gopls"`, `"rust-analyzer"`) is the internal server name used by Claude Code for logging and multi-server disambiguation. It does not need to match the binary name.

### Optional fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `args` | `string[]` | `[]` | CLI arguments passed to the server binary. Common: `["--stdio"]` for servers that don't default to stdio transport. |
| `transport` | `"stdio"` \| `"socket"` | `"stdio"` | Communication transport. `zls` uses stdio; do not change. |
| `env` | `object` | `{}` | Environment variables injected into the server process. Keys and values are strings. |
| `initializationOptions` | `object` | `{}` | Passed in the LSP `initialize` request's `initializationOptions` field. Server-specific. |
| `settings` | `object` | `{}` | Passed via `workspace/didChangeConfiguration`. Server-specific. Some servers read from `initializationOptions`, some from `settings`, some both. |
| `workspaceFolder` | `string` | (cwd) | Override the workspace root sent to the server. |
| `startupTimeout` | `number` | (unspecified) | Milliseconds to wait for server to respond to `initialize`. |
| `shutdownTimeout` | `number` | (unspecified) | Milliseconds to wait for graceful `shutdown` before SIGKILL. |
| `restartOnCrash` | `boolean` | (unspecified) | If `true`, restart the server process when it exits unexpectedly. |
| `maxRestarts` | `number` | (unspecified) | Maximum restart attempts before giving up. Caps `restartOnCrash`. |

### Schema drift from the user's reference doc

The fields in the pre-loaded reference are **accurate and current** as of this writing. No new fields were added between the reference doc date and 2026-04-26. The only clarification: `transport` defaults to `"stdio"` (not just `"stdio" | socket"`), and `workspaceFolder` defaults to the session's working directory.

### The `lspServers` key in `plugin.json`

You can inline the LSP config directly in `plugin.json` instead of a separate file:
```json
{
  "lspServers": {
    "zig": { ... }
  }
}
```
Or point to the file:
```json
{
  "lspServers": "./.lsp.json"
}
```
Both are equivalent. The official marketplace uses the separate-file pattern.

---

## Contribution Flow

### Path A — Official Anthropic marketplace (form-gated)

The official marketplace repo is `github.com/anthropics/claude-plugins-official`. It is **not an open PR-based contribution** like most open-source repos. The submission process is:

1. **Build and test your plugin** locally (see Local Testing section).
2. **Submit via in-app form**:
   - Claude.ai: `https://claude.ai/settings/plugins/submit`
   - Console: `https://platform.claude.com/plugins/submit`
3. Anthropic reviews for quality and security. External plugins land in `external_plugins/`, not `plugins/` (which is Anthropic-internal).
4. After approval, your plugin appears in the official marketplace under `external_plugins/zig-lsp/`.

**Naming convention**: `<language-server-binary>-lsp` (so: `zls-lsp` is acceptable, but `zig-lsp` also follows the pattern since `zls` is the Zig toolchain's name). Observe that `ruby-lsp` exists using language name rather than binary name — either convention is present.

**File requirements for submission** (from the README):
- `.claude-plugin/plugin.json` (required)
- `.lsp.json` at plugin root (required for LSP plugins — see Writing the Zig Plugin for why this matters)
- `README.md` (required — include binary install instructions)
- `LICENSE` (expected but not enforced)

### Path B — Community marketplace (immediate, no review)

Ship your own marketplace repo on GitHub:

```
your-github/zig-lsp-marketplace/
├── .claude-plugin/
│   └── marketplace.json     # Lists your plugin(s)
└── plugins/
    └── zig-lsp/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── .lsp.json
        └── README.md
```

`marketplace.json`:
```json
{
  "name": "zig-lsp-marketplace",
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "plugins": [
    {
      "name": "zig-lsp",
      "source": "./plugins/zig-lsp",
      "description": "Zig language support via ZLS"
    }
  ]
}
```

Users install with:
```shell
/plugin marketplace add yourgithub/zig-lsp-marketplace
/plugin install zig-lsp@zig-lsp-marketplace
```

Or as a flat single-plugin repo (simpler):
```shell
/plugin marketplace add yourgithub/zig-lsp
/plugin install zig-lsp@zig-lsp
```
(This works when the plugin directory IS the repo root and also contains `.claude-plugin/marketplace.json`.)

### Prior art: `4rgon4ut/cc-zig-lsp`

A community Zig plugin already exists at `github.com/4rgon4ut/cc-zig-lsp`. It ships with:
- Auto-download of `zls` if not found in PATH (shell hooks)
- Version detection from `build.zig.zon`, `.zigversion`, or `zig version`
- Multi-version caching at `~/.local/share/zls/`
- Active symlink at `~/.local/bin/zls`

For the official marketplace, a simpler plugin that assumes `zls` is user-installed (matching the Anthropic pattern) is likely preferred. The `4rgon4ut` plugin's approach is excellent for a community plugin.

---

## Writing the Zig Plugin

### `zls` operator facts

**Binary name**: `zls` (lowercase, no prefix). On Windows: `zls.exe`.

**Install methods** (user must do this — your plugin does not bundle it):

| Method | Command |
|--------|---------|
| ZVM (recommended) | `zvm i master --zls` |
| Zigup | `zigup 0.14.0 && zigup install-zls` |
| Prebuilt from zigtools.org | Download from `https://zigtools.org/zls/install/` |
| Build from source | `git clone https://github.com/zigtools/zls && zig build` |

**Default PATH locations**:
- ZVM: `~/.zvm/bin/zls` (add `~/.zvm/bin` to PATH)
- Zigup: `~/.local/bin/zls`
- Manual build: wherever you `zig build -p ~/.local`

**File extensions to map**:

| Extension | Language ID |
|-----------|-------------|
| `.zig` | `"zig"` |
| `.zon` | `"zig"` |

`.zon` (Zig Object Notation — used in `build.zig.zon` for package metadata) is handled by `zls` as Zig source. Map it to `"zig"`.

**LSP capabilities `zls` supports**:
- `textDocument/completion` (with snippets, argument placeholders)
- `textDocument/hover`
- `textDocument/definition` / `textDocument/declaration`
- `textDocument/documentSymbol`
- `textDocument/references`
- `textDocument/rename`
- `textDocument/formatting` (delegates to `zig fmt`)
- `textDocument/publishDiagnostics` (the critical one for Claude Code's auto-diagnostic hook)
- `textDocument/semanticTokens`
- `textDocument/inlayHint`
- `textDocument/codeAction`
- `textDocument/selectionRange`
- `textDocument/foldingRange`

**Known limitation**: `zls` cannot resolve complex comptime expressions. It provides parser-level diagnostics (syntax errors, unused variables) but type mismatch errors in comptime-heavy code require `build_on_save` to catch.

### Complete worked `.lsp.json`

```json
{
  "zig": {
    "command": "zls",
    "extensionToLanguage": {
      ".zig": "zig",
      ".zon": "zig"
    },
    "initializationOptions": {
      "enable_snippets": true,
      "enable_argument_placeholders": true,
      "enable_build_on_save": true,
      "build_on_save_args": [],
      "semantic_tokens": "full",
      "inlay_hints_show_variable_type_hints": true,
      "inlay_hints_show_parameter_name": true,
      "prefer_ast_check_as_child_process": true
    },
    "startupTimeout": 30000,
    "restartOnCrash": true,
    "maxRestarts": 3
  }
}
```

**Field-by-field rationale**:
- `enable_build_on_save: true` — enables compilation-backed diagnostics beyond parser errors. This is what makes `publishDiagnostics` actually useful in Claude Code's post-edit hook. Without it, only syntax errors appear.
- `prefer_ast_check_as_child_process: true` — default `true` in `zls`; `zig ast-check` catches more errors faster than the built-in checker.
- `semantic_tokens: "full"` — full semantic highlighting; `"partial"` is less expensive on large files.
- `startupTimeout: 30000` — 30 seconds; `zls` starts in under 1 second normally but give headroom for slow CI environments.
- `restartOnCrash: true` + `maxRestarts: 3` — `zls` is stable but actively developed; crash recovery prevents a broken session.

**Lean manifest** (matching the Anthropic minimal convention from PR #378):
```json
{
  "zig": {
    "command": "zls",
    "extensionToLanguage": {
      ".zig": "zig",
      ".zon": "zig"
    }
  }
}
```
`zls` defaults are reasonable. The fuller version above is recommended for contributor submissions because it makes defaults explicit for reviewers.

### `.claude-plugin/plugin.json`

```json
{
  "name": "zig-lsp",
  "description": "Zig language server for Claude Code (requires zls in PATH)",
  "version": "1.0.0",
  "author": {
    "name": "Your Name",
    "email": "you@example.com",
    "url": "https://github.com/yourusername"
  },
  "homepage": "https://github.com/yourusername/zig-lsp",
  "repository": "https://github.com/yourusername/zig-lsp",
  "license": "MIT",
  "keywords": ["zig", "zls", "lsp", "language-server", "code-intelligence"]
}
```

### `README.md` template

```markdown
# zig-lsp

Zig language server for Claude Code, providing code intelligence via [ZLS](https://github.com/zigtools/zls).

## Supported Extensions

`.zig`, `.zon`

## Installation

### 1. Install ZLS

Choose one method:

**Via ZVM (recommended):**
```
zvm i master --zls
```
Add `~/.zvm/bin` to your PATH.

**Via prebuilt binary:**
Download from https://zigtools.org/zls/install/ and place `zls` somewhere in your PATH.

**Verify installation:**
```
zls --version
```

### 2. Install the plugin

```
/plugin marketplace add <marketplace> && /plugin install zig-lsp@<marketplace>
```

Or for the official marketplace:
```
/plugin install zig-lsp@claude-plugins-official
```

## Build-on-save diagnostics (recommended)

For compilation-level error reporting beyond syntax errors, add a `check` step to your `build.zig`:

```zig
const exe_check = b.addExecutable(.{
    .name = "myapp",
    .root_source_file = b.path("src/main.zig"),
    .target = target,
    .optimize = optimize,
});
const check = b.step("check", "Check if myapp compiles");
check.dependOn(&exe_check.step);
```

ZLS will run `zig build check` (instead of emitting a binary) to get full diagnostics quickly.

## More Information

- [ZLS Documentation](https://zigtools.org/)
- [ZLS GitHub](https://github.com/zigtools/zls)
- [Zig Language](https://ziglang.org/)
```

---

## Local Testing Before PR

### Step 1 — Install `zls` and verify PATH

```bash
# After installing via your preferred method:
zls --version
# Should print: ZLS 0.x.y (or similar)

zls --show-config-path
# Prints the location of your zls.json config file (optional, not required by the plugin)
```

### Step 2 — Create the plugin directory

```bash
mkdir -p C:/Users/avife/zig-lsp-plugin/zig-lsp/.claude-plugin
# Create .lsp.json and plugin.json as shown in Writing the Zig Plugin
```

### Step 3 — Sideload with `--plugin-dir`

```bash
cd C:/Users/avife/zig-lsp-plugin
claude --plugin-dir ./zig-lsp
```

This loads the plugin for the session without installing it. When the same plugin name is also installed from a marketplace, the `--plugin-dir` version takes precedence (except managed plugins).

### Step 4 — Verify the plugin loaded

Inside Claude Code:
```
/plugin
```
Go to the **Installed** tab. Look for `zig-lsp`. If it shows errors, check the **Errors** tab.

Or validate the manifest:
```
/plugin validate
```

Or use the CLI before launching:
```bash
claude plugin validate ./zig-lsp
```

### Step 5 — Verify LSP is active

Check debug output:
```bash
claude --plugin-dir ./zig-lsp --debug
```

In the debug output, search for:
```
Total LSP servers loaded: 1
```
or
```
LSP notification handlers registered successfully for all 1 server(s)
```

If you see `0 server(s)`, the `.lsp.json` is not being picked up. Check: is `.lsp.json` at the plugin root (not inside `.claude-plugin/`)?

Enable LSP logging:
```bash
claude --plugin-dir ./zig-lsp --enable-lsp-logging
```
Logs appear at `~/.claude/debug/lsp-*.log`.

### Step 6 — Exercise the plugin

Open or create a `.zig` file in Claude Code's working directory:
```zig
// test.zig
const std = @import("std");
pub fn main() void {
    const x: u32 = "not a number"; // type error
    std.debug.print("hello\n", .{});
}
```

Ask Claude to edit it. After the edit, Claude Code should automatically receive diagnostics from `zls` via `textDocument/publishDiagnostics`. You should see the type error surface without running the compiler manually.

Test navigation:
> "What is the definition of `std.debug.print`?"

If Claude uses `textDocument/definition` (LSP-based) rather than `grep`, the plugin is working.

### Step 7 — Test with the marketplace path (before official submission)

```bash
# Create a local marketplace wrapper
mkdir -p ~/.local/my-zig-marketplace/.claude-plugin
cat > ~/.local/my-zig-marketplace/.claude-plugin/marketplace.json << 'EOF'
{
  "name": "my-zig-marketplace",
  "owner": { "name": "test" },
  "plugins": [
    {
      "name": "zig-lsp",
      "source": "/absolute/path/to/zig-lsp"
    }
  ]
}
EOF

# Add and install via marketplace path
/plugin marketplace add ~/.local/my-zig-marketplace
/plugin install zig-lsp@my-zig-marketplace
/reload-plugins
```

This tests the full installation path (file copying, cache, resolution) rather than just the `--plugin-dir` sideload.

---

## Pitfalls and Open Questions

### Known bugs as of 2026-04-26

**Bug: All 11 official LSP plugins were missing `.lsp.json`** (Issue #379)
- Status: PR #378 is open/merged to add `.lsp.json` to each plugin directory.
- Root cause: the official plugins only had `README.md`; the `lspServers` config existed only in `marketplace.json` but the plugin installer copies only the plugin `source` directory, not the marketplace entry's inline config.
- **Impact for contributors**: your new plugin MUST have `.lsp.json` as a physical file in the plugin directory. Relying on `lspServers` in `marketplace.json` alone is broken. Both the official Piebald community marketplace and the 4rgon4ut plugin correctly use `.lsp.json` at the plugin root.

**Bug: LSP not recognized despite correct config** (Issue #14803)
- Status: Closed, assigned to Anthropic.
- Symptoms: `"No LSP server available for file type: .zig"` even with correct structure.
- Workaround: ensure `claude --debug` shows `Total LSP servers loaded: 1`. If 0, check that `.lsp.json` is at root (not in `.claude-plugin/`), then run `/reload-plugins`.
- As of Claude Code v2.1+, this is largely fixed, but the debug-first workflow above remains the right test.

**Known LSP maturity note** (from the Piebald community):
> "LSP support in Claude Code is pretty raw still. There are bugs in the different LSP operations, no documentation, and no UI indication that your LSP servers are started/running/have errors or even exist."

The `/plugin` Errors tab is the primary diagnostic surface.

### `zls`-specific gotchas

- **`zls` version must match `zig` version exactly**. A mismatch causes the server to crash on startup. If `restartOnCrash: true` is set without a version match, you'll burn through `maxRestarts` on every session start. Advise users to install matching versions.
- **`enable_build_on_save` requires a `check` step in `build.zig`**. Without it, `zls` falls back to `zig ast-check` which only catches syntax and parser-level errors, not type errors. Your plugin's README should include the `build.zig` snippet from Writing the Zig Plugin.
- **`.zon` files**: the Zig package manifest (`build.zig.zon`) uses `.zon` extension. Map it in `extensionToLanguage`. Without this, opening `build.zig.zon` gets no LSP support.
- **Windows paths**: `zls.exe` is the binary on Windows. The `command` field should still be `"zls"` (no `.exe`) — the OS resolves the extension. Verify PATH includes the `zls` directory.
- **Cold-start on first open**: `zls` performs initial indexing on first file open. This is fast (seconds, not minutes like JVM servers), but the first `textDocument/didOpen` may be slow. `startupTimeout: 30000` is conservative — `zls` typically initializes in under 3 seconds.
- **Comptime**: `zls` cannot evaluate complex `comptime` expressions. This is a known limitation. Build-on-save covers the gap. Document this in your README.

### Open questions to raise with Anthropic maintainers

1. **Will PR #378 be merged before the next stable release?** If not, contributors should follow the file-in-directory pattern now and not rely on marketplace.json inline `lspServers`.
2. **Is `ruby-lsp` shipping alongside the 11 existing plugins?** The directory listing showed `ruby-lsp` in `plugins/`. If so, the official count is 12, not 11.
3. **Naming: `zig-lsp` or `zls-lsp`?** The convention is inconsistent: `gopls-lsp` uses the binary name, `csharp-lsp` uses the language name. Ask in the submission form which is preferred. `zig-lsp` is more discoverable; `zls-lsp` matches the `gopls-lsp`/`rust-analyzer-lsp` pattern.
4. **Is `initializationOptions` or `settings` the right field for `zls` config?** The `zls` schema.json defines settings as `initializationOptions`-delivered. Verify this against the server logs during testing — some LSP servers read from `workspace/didChangeConfiguration` (`settings` field) instead.
5. **Platform support**: the official Anthropic plugin system does not scope plugins by OS. If `zls` binary install paths differ by platform, document all of them in the README but do not try to conditionally set `command` in `.lsp.json` (it doesn't support per-platform values).

---

## References

| Resource | URL |
|----------|-----|
| Claude Code Plugins Reference — LSP Servers | [code.claude.com/docs/en/plugins-reference](https://code.claude.com/docs/en/plugins-reference#lsp-servers) |
| Create Plugins — Official Guide | [code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins) |
| Discover and Install Plugins | [code.claude.com/docs/en/discover-plugins](https://code.claude.com/docs/en/discover-plugins) |
| Create and Distribute a Marketplace | [code.claude.com/docs/en/plugin-marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) |
| anthropics/claude-plugins-official | [github.com/anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) |
| PR #378 — Add .lsp.json to all LSP plugins | [github.com/anthropics/claude-plugins-official/pull/378](https://github.com/anthropics/claude-plugins-official/pull/378) |
| Issue #379 — All LSP plugins missing .lsp.json | [github.com/anthropics/claude-plugins-official/issues/379](https://github.com/anthropics/claude-plugins-official/issues/379) |
| Issue #14803 — LSP plugins not recognized | [github.com/anthropics/claude-code/issues/14803](https://github.com/anthropics/claude-code/issues/14803) |
| cc-zig-lsp — community Zig plugin | [github.com/4rgon4ut/cc-zig-lsp](https://github.com/4rgon4ut/cc-zig-lsp) |
| Piebald-AI/claude-code-lsps | [github.com/Piebald-AI/claude-code-lsps](https://github.com/Piebald-AI/claude-code-lsps) |
| zigtools/zls | [github.com/zigtools/zls](https://github.com/zigtools/zls) |
| ZLS Installation Guide | [zigtools.org/zls/install/](https://zigtools.org/zls/install/) |
| ZLS schema.json — all config keys | [github.com/zigtools/zls/blob/master/schema.json](https://github.com/zigtools/zls/blob/master/schema.json) |
| Improving Your ZLS Experience (Loris Cro) | [kristoff.it/blog/improving-your-zls-experience/](https://kristoff.it/blog/improving-your-zls-experience/) |
| Using Claude Code LSP Without Official Marketplace | [dev.classmethod.jp/en/articles/claude-code-lsp-from-local-marketplace/](https://dev.classmethod.jp/en/articles/claude-code-lsp-from-local-marketplace/) |
| DataCamp — How to Build Claude Code Plugins | [datacamp.com/tutorial/how-to-build-claude-code-plugins](https://www.datacamp.com/tutorial/how-to-build-claude-code-plugins) |
| zircote/lsp-marketplace | [github.com/zircote/lsp-marketplace](https://github.com/zircote/lsp-marketplace) |
| claude-code/plugins README | [github.com/anthropics/claude-code/blob/main/plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) |

---

*This guide was synthesized from 24 sources. See `resources/claude-code-lsp-plugin-contribution-sources.json` for full source metadata.*
