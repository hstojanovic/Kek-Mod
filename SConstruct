import json
import re
import shutil
import subprocess as sp
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path

# SConstruct_config.py is gitignored and machine-specific. Copy
# SConstruct_config.py.example to SConstruct_config.py and set your local paths.
try:
    import SConstruct_config as cfg
except ImportError:
    msg = (
        'SConstruct_config.py not found. Copy SConstruct_config.py.example to '
        'SConstruct_config.py and set your local paths (see that file for the variables).'
    )
    raise SystemExit(msg) from None
from SCons.Script import *

SetOption('num_jobs', cfg.NUM_JOBS)

build = ARGUMENTS.get('build', 'Release').title()
if build not in {'Debug', 'Release', 'Profile', 'Final'}:
    msg = f"Invalid build target '{build}', choose 'debug', 'release', 'profile' or 'final'"
    raise SystemExit(msg)

DLL_SRC = 'CvGameCoreDLL'
BUILD_DIR = f'{DLL_SRC}/build/{build}'

env = Environment(
    tools=['default', 'compilation_db'],
    CPPPATH=[
        f'{cfg.BTS_SRC}/Boost-1.32.0/include',
        f'{cfg.BTS_SRC}/Python24/include',
        f'{cfg.VS_TOOLKIT_03}/include',
        f'{cfg.WIN_SDK}/Include',
        f'{cfg.WIN_SDK}/Include/mfc',
        f'{cfg.VS_TOOLKIT_10}/include',
    ],
    LIBPATH=[
        f'{cfg.BTS_SRC}/Python24/libs',
        f'{cfg.BTS_SRC}/Boost-1.32.0/libs',
        f'{cfg.VS_TOOLKIT_03}/lib',
        f'{cfg.WIN_SDK}/Lib',
    ],
    LIBS=['winmm', 'user32', 'boost_python-vc71-mt-1_32'],
    CPPFLAGS=['/W3', '/GS', '/GR', '/Gy', '/EHsc', '/Gd', '/Gm-', '/MD'],
    CPPDEFINES=['WIN32', '_WINDOWS', '_USRDLL'],
    LINKFLAGS=['/DLL', '/NOLOGO', '/SUBSYSTEM:WINDOWS', '/LARGEADDRESSAWARE'],
    CC=f'"{cfg.VS_TOOLKIT_03}/bin/cl.exe"',
    CXX=f'"{cfg.VS_TOOLKIT_03}/bin/cl.exe"',
    LINK=f'"{cfg.VS_TOOLKIT_03}/bin/link.exe"',
    RC=f'"{cfg.WIN_SDK}/bin/rc.exe"',
    COMPILATIONDB_USE_ABSPATH=True,
)

if build == 'Debug':
    env.Append(
        CPPDEFINES=['_DEBUG', 'LOG_AI'],
        CPPFLAGS=['/RTC1', '/Z7', '/Od'],
        LIBS=['msvcprt'],
        LINKFLAGS=['/INCREMENTAL', '/DEBUG'],
    )
if build in {'Release', 'Profile', 'Final'}:
    env.Append(
        CPPDEFINES=['NDEBUG', 'FINAL_RELEASE'],
        CPPFLAGS=['/O2', '/Oy', '/Oi', '/Og', '/G7', '/arch:SSE2'],
        LINKFLAGS=['/INCREMENTAL:NO', '/OPT:REF', '/OPT:ICF'],
    )
if build == 'Profile':
    env.Append(CPPDEFINES=['FP_PROFILE_ENABLE', 'USE_INTERNAL_PROFILER'])
# 'Final' is Release + whole-program optimization (/GL + /LTCG): a slower link,
# for shipped releases only. Kept off Release (fast iteration) and Profile (LTO
# inlines away the call boundaries USE_INTERNAL_PROFILER measures).
if build == 'Final':
    env.Append(CPPFLAGS=['/GL'], LINKFLAGS=['/LTCG'])

VariantDir(BUILD_DIR, DLL_SRC, duplicate=False)
# _precompile.cpp is built separately as the PCH; CvTextScreens.cpp ships in source but isn't part of the DLL.
src = [file for file in Glob(f'{BUILD_DIR}/*.cpp') if file.name not in {'_precompile.cpp', 'CvTextScreens.cpp'}]

# Built-in PCH integration: env['PCH'] makes SCons add /Yu, /Fp, and link the
# corresponding .obj automatically for every compile in this env.
pch_files = env.PCH(target=f'{BUILD_DIR}/_precompile.pch', source=f'{BUILD_DIR}/_precompile.cpp')
env['PCH'] = pch_files[0]
env['PCHSTOP'] = 'CvGameCoreDLL.h'

if build in {'Release', 'Final'}:
    res_builder = Builder(action='$RC /fo $TARGET $RCFLAGS $SOURCES')
    env.Append(BUILDERS={'Res': res_builder})
    res = env.Res(
        target=f'{BUILD_DIR}/CvGameCoreDLL.res',
        source=f'{BUILD_DIR}/CvGameCoreDLL.rc',
        RCFLAGS=f'/I"{cfg.WIN_SDK}/Include"',
    )
    src += res

dll = env.SharedLibrary(target=f'{BUILD_DIR}/CvGameCoreDLL', source=src)
Clean(BUILD_DIR, ['*.dll', '*.exp', '*.idb', '*.ilk', '*.lib', '*.obj', '*.pch', '*.pdb', '*.res'])

# Emit compile_commands.json for VS Code IntelliSense.
compdb = env.CompilationDatabase('compile_commands.json')


# Convert compile_commands.json `command` strings to `arguments` arrays and
# rewrite vendor /I includes to -isystem. Arguments-array form skips shell
# tokenization so paths with apostrophes (Sid Meier's...) and spaces resolve
# cleanly. -isystem makes clang auto-suppress warnings from those headers.
# MSVC's cl.exe still uses /I for the real build; we only rewrite the
# IDE-facing JSON.
_VENDOR_FRAGMENTS = (
    'Boost-1.32.0',
    'Python24',
    'Microsoft_Visual_C++_Toolkit_2003',
    'Microsoft Visual Studio 10.0',
    'WindowsSDK',
)
# /I forms SCons emits: "/I<path>", /I"<path>", /I<path>.
_INCLUDE_RE = re.compile(r'^"?/I"?([^"]+)"?$')
# MSVC PCH / output flags clang-cl can't honor (binary PCH it can't read,
# output paths irrelevant to AST parsing). /FI is kept — that's our own
# force-include preamble, separate from these.
_STRIP_PREFIXES = ('/Yu', '/Yc', '/Fp', '/Fo', '/Fd')


def tokenize_windows(s: str) -> list[str]:
    # Minimal Windows-style command-line tokenizer: splits on whitespace,
    # respects double-quoted regions, keeps embedded apostrophes literal.
    tokens: list[str] = []
    buf: list[str] = []
    in_quotes = False
    for ch in s:
        if ch == '"':
            in_quotes = not in_quotes
            continue
        if ch.isspace() and not in_quotes:
            if buf:
                tokens.append(''.join(buf))
                buf = []
            continue
        buf.append(ch)
    if buf:
        tokens.append(''.join(buf))
    return tokens


# clang-only compile flags injected into compile_commands.json. The real
# cl.exe build doesn't read this file — only tools that consume it (clangd,
# clang-tidy, clang-cl --analyze) pick these up. Single source of truth.
#
# Position-sensitive: must land BEFORE the source file so they precede
# clang's target-macro predefines. .clangd's Add: appends after the source
# and is too late for --target in particular (see clangd #555).
_PREAMBLE_FLAGS = (
    # Override clangd's x64 host default; otherwise _WIN64 gets defined and
    # MSVC headers take the wrong branches.
    '--target=i386-pc-windows-msvc',
    '-std=c++03',
    '-fms-extensions',
    '-fms-compatibility',
    # VC++ 7.1 lineage.
    '-fms-compatibility-version=13.10',
    '-Wall',
    # Resolve quoted #includes when parsing a header without a .cpp driver.
    '-ICvGameCoreDLL',
    # K-Mod's ART_INFO_DECL et al. rely on MSVC's relaxed ## concatenation;
    # clang warns on the non-token results even with -fms-extensions.
    '-Wno-invalid-token-paste',
)
# stdint.h is the canonical C99 header for intptr_t / uintptr_t. clang ships
# its own stdint.h that declares these from clang's internal type macros
# (correctly sized for the target). VC++ 7.1's stddef.h would declare them too,
# but clang's bundled stddef.h shadows MSVC's on Windows and doesn't expose
# them — see FLTK #752 for the same issue. Force-including stdint.h once
# up front fills the C99 types hole regardless of include order downstream.
_FORCE_INCLUDES = ('stdint.h', 'CvGameCoreDLL/CvGameCoreDLL.h')


def rewrite_vendor_to_isystem(target: list, source: list, env: object) -> None:
    path = Path(str(target[0]))
    db = json.loads(path.read_text(encoding='utf-8'))

    for entry in db:
        args = tokenize_windows(entry['command'])
        rewritten: list[str] = []
        for arg in args:
            if arg.startswith(_STRIP_PREFIXES):
                continue
            match = _INCLUDE_RE.match(arg)
            if match:
                inc_path = match.group(1)
                if any(frag in inc_path for frag in _VENDOR_FRAGMENTS):
                    # clang-cl's native system-include flag; equivalent to
                    # -isystem but more reliable in clang-cl driver mode.
                    rewritten.extend(('/imsvc', inc_path))
                    continue
                rewritten.append(f'/I{inc_path}')
                continue
            rewritten.append(arg)

        # Insert preamble flags + force-includes right after the compiler
        # path (argv[0]) so they precede any /c <file> argument.
        # /FI<file> (joined, no space) is clang-cl's native force-include flag.
        # Using `-include FILE` (2 args) doesn't survive clang-cl's argument
        # reordering: anything ending in .h gets classified as a source file
        # and moved to after `--`, leaving naked -include flags.
        prefix = [rewritten[0]]
        prefix.extend(_PREAMBLE_FLAGS)
        prefix.extend(f'/FI{inc}' for inc in _FORCE_INCLUDES)
        entry['arguments'] = prefix + rewritten[1:]
        del entry['command']

    path.write_text(json.dumps(db, indent=4), encoding='utf-8')


env.AddPostAction(compdb, rewrite_vendor_to_isystem)
Default(dll, compdb)


# `uv run scons gen_vscode` writes .vscode/launch.json from cfg values.
LAUNCH_JSON = Path(Dir('#').abspath) / '.vscode' / 'launch.json'


def write_vscode_launch(target: list, source: list, env: object) -> None:
    config = {
        'version': '0.2.0',
        'configurations': [
            {
                'name': 'Launch Civ4 BTS',
                'type': 'cppvsdbg',
                'request': 'launch',
                'program': cfg.GAME_EXE,
                'args': [f'mod="{cfg.MOD_NAME}"'],
                'cwd': str(Path(cfg.GAME_EXE).parent),
                'console': 'integratedTerminal',
            }
        ],
    }
    LAUNCH_JSON.parent.mkdir(parents=True, exist_ok=True)
    LAUNCH_JSON.write_text(json.dumps(config, indent=4) + '\n', encoding='utf-8')


launch_cmd = env.Command(str(LAUNCH_JSON), [], write_vscode_launch)
env.AlwaysBuild(launch_cmd)
env.Alias('gen_vscode', [launch_cmd, compdb])


# `uv run scons deploy` mirrors mod content + freshly-built DLL
# to <BTS>/Mods/<MOD_NAME>/. Mirror is timestamp-skip with deletes (rsync-like).
REPO_ROOT = Path(Dir('#').abspath)
DEPLOY_INCLUDE = (
    'Assets',
    'Info',
    'PublicMaps',
    'Settings',
    'changelog.txt',
    'readme.txt',
    'K-Mod changelog.txt',
    'K-Mod readme.txt',
    'Kek-Mod.bat',
    'Kek-Mod.ini',
    'update_config.json',
    'update_info.json',
)


def mirror(src_dir: Path, dst_dir: Path, exempt: frozenset[str] = frozenset()) -> None:
    dst_dir.mkdir(parents=True, exist_ok=True)
    src_entries = {p.name: p for p in src_dir.iterdir()}
    for name, src in src_entries.items():
        dst = dst_dir / name
        if src.is_dir():
            mirror(src, dst)
        elif not dst.exists() or src.stat().st_mtime > dst.stat().st_mtime:
            shutil.copy2(src, dst)
    for dst in dst_dir.iterdir():
        if dst.name in src_entries or dst.name in exempt:
            continue
        if dst.is_dir():
            shutil.rmtree(dst)
        else:
            dst.unlink()


def do_deploy(target: list, source: list, env: object) -> None:
    dest_root = Path(cfg.GAME_EXE).parent / 'Mods' / cfg.MOD_NAME
    print(f'Deploying to {dest_root}')
    for item in DEPLOY_INCLUDE:
        src = REPO_ROOT / item
        if not src.exists():
            continue
        dst = dest_root / item
        if src.is_dir():
            # Don't sweep the deployed DLL out of <dest>/Assets/.
            exempt = frozenset({'CvGameCoreDLL.dll'} if item == 'Assets' else ())
            mirror(src, dst, exempt)
        elif not dst.exists() or src.stat().st_mtime > dst.stat().st_mtime:
            dst.parent.mkdir(parents=True, exist_ok=True)
            shutil.copy2(src, dst)
    dll_src = Path(str(source[0].abspath))
    dll_dst = dest_root / 'Assets' / 'CvGameCoreDLL.dll'
    shutil.copy2(dll_src, dll_dst)
    print(f'Deployed {build} DLL -> {dll_dst}')
    Path(str(target[0])).touch()


deploy_stamp = env.Command(f'{BUILD_DIR}/.deploy.stamp', dll, do_deploy)
env.AlwaysBuild(deploy_stamp)
env.NoClean(deploy_stamp)
env.Alias('deploy', deploy_stamp)


# Code quality passes: `scons format` / `scons tidy` / `scons analyze`.
# clang-format / clang-tidy / clang-cl assumed on PATH.
# Stamps land under build/ (gitignored) so SCons can track invocation state.
TOOLS_STAMP_DIR = f'{DLL_SRC}/build'


def list_dll_sources() -> list[Path]:
    src_dir = Path(DLL_SRC)
    return list(src_dir.glob('*.cpp')) + list(src_dir.glob('*.h'))


def _tool(name: str) -> str:
    # Resolve a build tool to its absolute path: clearer error than a raw
    # subprocess failure if it's missing, and avoids partial-path execution.
    path = shutil.which(name)
    if path is None:
        msg = f'{name} not found on PATH'
        raise SystemExit(msg)
    return path


def _jj_diff(base: str) -> str:
    # `jj diff` from <base> to the working copy, with a friendly error if jj is
    # missing or the revision is bad (line-scoped format needs jj; all=1 doesn't).
    jj = shutil.which('jj')
    if jj is None:
        msg = (
            'scons format needs jj on PATH for line-scoped formatting. Install jj, '
            'or run `scons format all=1` to reformat the whole tree.'
        )
        raise SystemExit(msg)
    proc = sp.run([jj, 'diff', '--git', '--from', base, '--to', '@'], capture_output=True, text=True, check=False)
    if proc.returncode != 0:
        msg = (
            f'jj diff failed (is base={base!r} a valid revision?):\n'
            f'{proc.stderr.strip()}\n'
            'Run `scons format all=1` to reformat the whole tree instead.'
        )
        raise SystemExit(msg)
    return proc.stdout


def _added_lines(diff: str) -> dict[str, list[int]]:
    # Map each changed DLL .cpp/.h to its added line numbers in the new (working-
    # copy) file. jj emits whole-file hunks, so track new-file line numbers per
    # line marker rather than trusting hunk spans.
    added: dict[str, list[int]] = {}
    path = None
    new_lineno = 0
    for line in diff.splitlines():
        if line.startswith('diff --git'):
            path = None
        elif line.startswith('+++ '):
            name = line[4:].strip().removeprefix('b/')
            if name.startswith(f'{DLL_SRC}/') and name.endswith(('.cpp', '.h')):
                path = name
                added.setdefault(path, [])
            else:
                path = None
        elif line.startswith('@@'):
            match = re.search(r'\+(\d+)', line)
            new_lineno = int(match.group(1)) if match else 1
        elif path is not None and line[:1] == '+':
            added[path].append(new_lineno)
            new_lineno += 1
        elif path is not None and line[:1] == ' ':
            new_lineno += 1
        # '-' (removed) and '\' (no-newline) lines don't advance the new-file counter.
    return added


def changed_line_ranges(base: str) -> dict[str, list[list[int]]]:
    # Collapse the changed lines (vs <base>) into [start, end] ranges per file, so
    # we format only the lines we touched — reformatting never churns untouched
    # upstream code, which would wreck the next K-Mod rebase.
    ranges: dict[str, list[list[int]]] = {}
    for name, nums in _added_lines(_jj_diff(base)).items():
        spans: list[list[int]] = []
        for n in sorted(set(nums)):
            if spans and n == spans[-1][1] + 1:
                spans[-1][1] = n
            else:
                spans.append([n, n])
        if spans:
            ranges[name] = spans
    return ranges


def do_format(target: list, source: list, env: object) -> None:
    # Default: format only lines changed since `base` (default @-, the current
    # change), so untouched upstream code is left alone and K-Mod rebases stay
    # clean. `scons format base=<rev>` widens the scope; `scons format all=1`
    # reformats the whole tree.
    clang_format = _tool('clang-format')
    if ARGUMENTS.get('all'):
        files = [str(f) for f in list_dll_sources()]
        print(f'clang-format: {len(files)} files')
        # Batch to stay under Windows' ~32 KB CreateProcess command-line limit.
        batch = 40
        for i in range(0, len(files), batch):
            sp.run([clang_format, '-i', *files[i : i + batch]], check=True)
    else:
        base = ARGUMENTS.get('base', '@-')
        ranges = changed_line_ranges(base)
        if not ranges:
            print(f'clang-format: no changed C++ lines since {base}')
        for name, spans in ranges.items():
            line_args = [f'--lines={a}:{b}' for a, b in spans]
            print(f'clang-format: {name} ({len(spans)} range(s))')
            sp.run([clang_format, '-i', *line_args, name], check=True)
    Path(str(target[0])).touch()


def do_tidy(target: list, source: list, env: object) -> None:
    db = json.loads(Path('compile_commands.json').read_text(encoding='utf-8'))
    files = [entry['file'] for entry in db]
    clang_tidy = _tool('clang-tidy')
    print(f'clang-tidy: {len(files)} TUs')

    def run_one(file: str) -> tuple[str, str]:
        result = sp.run([clang_tidy, '-p', '.', '-quiet', file], capture_output=True, text=True, check=False)
        return file, result.stdout

    with ThreadPoolExecutor(max_workers=cfg.NUM_JOBS) as ex:
        for file, out in ex.map(run_one, files):
            if out.strip():
                print(f'\n=== {file} ===')
                print(out)
    Path(str(target[0])).touch()


def do_analyze(target: list, source: list, env: object) -> None:
    db = json.loads(Path('compile_commands.json').read_text(encoding='utf-8'))
    clang_cl = _tool('clang-cl')
    print(f'clang --analyze: {len(db)} TUs')

    def run_one(entry: dict) -> tuple[str, str]:
        # Replace argv[0] (cl.exe path) with clang-cl; PCH/output flags already
        # stripped, MSVC compat + --target etc. already baked in via the
        # rewrite_vendor_to_isystem pass.
        cmd = [clang_cl, *entry['arguments'][1:], '--analyze', '-Xclang', '-analyzer-output=text']
        result = sp.run(cmd, capture_output=True, text=True, check=False)
        return entry['file'], (result.stdout + result.stderr).strip()

    with ThreadPoolExecutor(max_workers=cfg.NUM_JOBS) as ex:
        for file, output in ex.map(run_one, db):
            if output:
                print(f'\n=== {file} ===')
                print(output)
    Path(str(target[0])).touch()


format_stamp = env.Command(f'{TOOLS_STAMP_DIR}/.format.stamp', [], do_format)
env.AlwaysBuild(format_stamp)
env.NoClean(format_stamp)
env.Alias('format', format_stamp)

tidy_stamp = env.Command(f'{TOOLS_STAMP_DIR}/.tidy.stamp', compdb, do_tidy)
env.AlwaysBuild(tidy_stamp)
env.NoClean(tidy_stamp)
env.Alias('tidy', tidy_stamp)

analyze_stamp = env.Command(f'{TOOLS_STAMP_DIR}/.analyze.stamp', compdb, do_analyze)
env.AlwaysBuild(analyze_stamp)
env.NoClean(analyze_stamp)
env.Alias('analyze', analyze_stamp)
