# Building Kek-Mod

This covers building the C++ game-core DLL (`CvGameCoreDLL/`). Editing XML,
Python, art, etc. under `Assets/` requires no build — only the DLL does.

The DLL is **32-bit, C++03**, compiled with the **Microsoft Visual C++ Toolkit
2003** (`cl.exe` 7.1) and driven by **SCons**, which is in turn driven by
[`uv`](https://docs.astral.sh/uv/). Clang (`clang-cl` / `clangd` /
`clang-format` / `clang-tidy`) is used **only** for editor IntelliSense and the
optional code-quality passes — never for the shipped build.

## Prerequisites

| Tool / path | Used for | Config variable |
| --- | --- | --- |
| [`uv`](https://docs.astral.sh/uv/) | Drives SCons via `uv run scons …`; installs the pinned SCons dev dependency (`pyproject.toml`) and Python 3.14 (`.python-version`) on first run | — |
| Microsoft Visual C++ Toolkit 2003 | `cl.exe` / `link.exe` (the real compiler/linker) | `VS_TOOLKIT_03` |
| Microsoft Visual Studio 10.0 / VC | Additional C++ headers | `VS_TOOLKIT_10` |
| Windows SDK (with `Include/`, `Lib/`, `bin/rc.exe`) | Headers, libs, resource compiler | `WIN_SDK` |
| A Civ4 BTS install's `CvGameCoreDLL/` | Provides `Boost-1.32.0/` and `Python24/` | `BTS_SRC` |
| Civ4 BTS executable | Debug-launch target and deploy destination | `GAME_EXE` |
| LLVM (`clang-cl`, `clang-format`, `clang-tidy` on `PATH`) | Optional: `format` / `tidy` / `analyze` targets and IntelliSense | — |

## One-time setup

Copy the config template and fill in your local paths:

```sh
cp SConstruct_config.py.example SConstruct_config.py
```

`SConstruct_config.py` is gitignored and machine-specific — never commit it.
Edit it to point at the tools and paths above (`MOD_NAME` is the mod folder name
used by `deploy` and `gen_vscode`).

## Building from the command line

All commands run through `uv`, which provides SCons without a manual install:

```sh
# Build the DLL (Release by default) + compile_commands.json for IntelliSense
uv run scons

# Pick a configuration: debug | release | profile | final
uv run scons build=debug

# Build the selected config and deploy the whole mod (incl. the DLL) to
# <BTS install>/Mods/<MOD_NAME>/ (use build=final for a shipped release)
uv run scons build=release deploy

# Write .vscode/launch.json from GAME_EXE / MOD_NAME (run once, or when they change)
uv run scons gen_vscode

# Clean a configuration's build artifacts
uv run scons build=release -c
```

The built DLL lands at `CvGameCoreDLL/build/<Config>/CvGameCoreDLL.dll`
(not committed; `CvGameCoreDLL/build/` is gitignored). `deploy` copies it to
`<BTS install>/Mods/<MOD_NAME>/Assets/CvGameCoreDLL.dll` alongside the mod
content. The CLI default configuration is **Release**.

**Configurations:** `debug` (asserts, unoptimized), `release` (optimized, no LTO — the dev default), `profile` (release + the internal profiler), and `final` (release + whole-program optimization, `/GL` + `/LTCG`). `final` produces a faster DLL but links much more slowly, so use it only for shipped releases; `release`/`profile` omit LTO for fast iteration — and `profile` must, so the function-call boundaries the profiler measures survive.

## Building from VS Code

`.vscode/tasks.json` wraps the same commands. **Ctrl+Shift+B** runs **Build**;
the other tasks (Build / Clean / Generate launch.json / Deploy) are available via
*Terminal → Run Task*. Each prompts for a build configuration (the VS Code tasks
default to **debug**). After running **Generate launch.json**, **F5** launches
Civ4 BTS with this mod under the Visual Studio debugger (`cppvsdbg`).

IntelliSense is provided by `clangd` reading `compile_commands.json` (emitted by
any build). See `.clangd`, `.clang-tidy`, and `.clang-format` for the
editor/analysis configuration.

## Code-quality passes (optional)

Require `clang-format` / `clang-tidy` / `clang-cl` on `PATH`:

```sh
uv run scons tidy      # clang-tidy over all translation units
uv run scons analyze   # clang-cl --analyze over all translation units
uv run scons format    # clang-format only the LINES you changed
```

> By default `format` reformats only the lines changed since the current change's
> parent (`@-`), so it leaves untouched upstream code alone (reformatting it would
> conflict on the next K-Mod rebase). Widen the scope with `scons format base=<rev>`
> (e.g. `base=DLP_0.3.1` for everything since the last release), or reformat the
> whole tree with `scons format all=1`. Line-scoped mode needs `jj` on `PATH`;
> `all=1` does not.

## Migrating from the old MSVC project

The Visual Studio solution/project files (`CvGameCoreDLL.sln`,
`CvGameCoreDLL.vcxproj*`) have been removed. "Open the solution in Visual
Studio" no longer applies — use `uv run scons …` or the VS Code
tasks above.
