# zig-lsp

Zig language server for Claude Code, providing code intelligence via [ZLS](https://github.com/zigtools/zls).

## Supported Extensions

`.zig`, `.zon`

## Installation

### 1. Install ZLS

Pick one method. ZLS must be on your `PATH` as `zls`.

**Via [ZVM](https://github.com/tristanisham/zvm) (recommended — keeps `zls` and `zig` in lockstep):**

```sh
zvm i master --zls
```

Add `~/.zvm/bin` to your `PATH`.

**Via prebuilt binary:**

Download from <https://zigtools.org/zls/install/> and place `zls` somewhere on your `PATH`.

**Build from source:**

```sh
git clone https://github.com/zigtools/zls
cd zls
zig build -Doptimize=ReleaseSafe
```

**Verify installation:**

```sh
zls --version
```

### 2. Install the plugin

```
/plugin marketplace add agent-sh/zig-lsp
/plugin install zig-lsp@agent-sh
```

After install, open a `.zig` file. Claude Code will report `zls` diagnostics automatically after every `Write` / `Edit` and can call `LSP` operations (definition, references, hover) directly.

## Build-on-save diagnostics

The plugin sets `enable_build_on_save: true`. To get compilation-level diagnostics (beyond syntax / parser errors), add a `check` step to your `build.zig` so ZLS can run `zig build check` without producing artifacts:

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

Without this step, ZLS falls back to `zig ast-check`, which catches syntax and parser errors only — no type errors.

## Configuration

The plugin ships sensible defaults. To override, edit `.lsp.json` in your local plugin directory or set workspace-level options in your project's `zls.json`. See the [ZLS schema](https://github.com/zigtools/zls/blob/master/schema.json) for the full list.

Defaults set by this plugin:

| Option | Value | Why |
| --- | --- | --- |
| `enable_snippets` | `true` | Snippet completions |
| `enable_argument_placeholders` | `true` | Tabstops in function-call snippets |
| `enable_build_on_save` | `true` | Real type diagnostics — see above |
| `semantic_tokens` | `"full"` | Highlighting / structure for navigation |
| `inlay_hints_show_variable_type_hints` | `true` | Inline types |
| `inlay_hints_show_parameter_name` | `true` | Inline parameter names at call sites |
| `prefer_ast_check_as_child_process` | `true` | Faster, more accurate parser-level errors |
| `startupTimeout` | `30000` ms | Conservative — ZLS usually starts in under 3 s |
| `restartOnCrash` | `true` | Recover from rare crashes mid-session |
| `maxRestarts` | `3` | Cap restart loop on persistent failures |

## Gotchas

- **`zls` and `zig` must be the same version.** A mismatch crashes ZLS on startup. With `restartOnCrash: true` you'll burn through `maxRestarts` per session. ZVM keeps them in lockstep — recommended.
- **`comptime`-heavy code:** ZLS cannot fully evaluate complex `comptime` expressions. The build-on-save step covers most of this gap.
- **`build.zig.zon`:** the package manifest is mapped to the `zig` language so you get LSP support inside it. Without this mapping, `.zon` files would be opaque.
- **Windows:** the binary is `zls.exe`, but the `command` field is just `"zls"` — the OS resolves the extension. Make sure the directory containing `zls.exe` is on `PATH`.

## Troubleshooting

If LSP features don't activate after install:

```sh
claude --debug
```

Look for `Total LSP servers loaded:` in the output. If `0`, the plugin didn't pick up `.lsp.json`.

Enable LSP-specific logs:

```sh
claude --enable-lsp-logging
```

Logs land at `~/.claude/debug/lsp-*.log`.

Inside Claude Code, the `/plugin` panel's **Installed** and **Errors** tabs are the primary diagnostic surface.

## Links

- [ZLS](https://github.com/zigtools/zls)
- [ZLS install guide](https://zigtools.org/zls/install/)
- [Improving Your ZLS Experience — Loris Cro](https://kristoff.it/blog/improving-your-zls-experience/)
- [Zig](https://ziglang.org/)
- [Claude Code plugin reference](https://code.claude.com/docs/en/plugins-reference#lsp-servers)

## License

MIT — see [LICENSE](../../LICENSE).
