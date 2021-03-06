project(
    'libxkbcommon',
    'c',
    version: '0.10.0',
    default_options: [
        'c_std=c99',
        'warning_level=2',
        'b_lundef=true',
    ],
    meson_version : '>= 0.41.0',
)
pkgconfig = import('pkgconfig')
cc = meson.get_compiler('c')


# Compiler flags.
foreach cflag: [
    '-fvisibility=hidden',
    '-fno-strict-aliasing',
    '-fsanitize-undefined-trap-on-error',
    '-Wextra',
    '-Wno-unused-parameter',
    '-Wno-missing-field-initializers',
    '-Wpointer-arith',
    '-Wmissing-declarations',
    '-Wformat=2',
    '-Wstrict-prototypes',
    '-Wmissing-prototypes',
    '-Wnested-externs',
    '-Wbad-function-cast',
    '-Wshadow',
    '-Wlogical-op',
    '-Wdate-time',
    '-Wwrite-strings',
    '-Wno-documentation-deprecated-sync',
]
    if cc.has_argument(cflag)
        add_project_arguments(cflag, language: 'c')
    endif
endforeach


# The XKB config root.
XKBCONFIGROOT = get_option('xkb-config-root')
if XKBCONFIGROOT == ''
    xkeyboard_config_dep = dependency('xkeyboard-config', required: false)
    if xkeyboard_config_dep.found()
        XKBCONFIGROOT = xkeyboard_config_dep.get_pkgconfig_variable('xkb_base')
    else
        XKBCONFIGROOT = join_paths(get_option('prefix'), get_option('datadir'), 'X11', 'xkb')
  endif
endif


# The X locale directory for compose.
XLOCALEDIR = get_option('x-locale-root')
if XLOCALEDIR == ''
    XLOCALEDIR = join_paths(get_option('prefix'), get_option('datadir'), 'X11', 'locale')
endif


# config.h.
configh_data = configuration_data()
# Like AC_USE_SYSTEM_EXTENSIONS, what #define to use to get extensions
# beyond the base POSIX function set.
if host_machine.system() == 'sunos'
  system_extensions = '__EXTENSIONS__'
else
  system_extensions = '_GNU_SOURCE'
endif
configh_data.set(system_extensions, 1)
system_ext_define = '#define ' + system_extensions
configh_data.set_quoted('DFLT_XKB_CONFIG_ROOT', XKBCONFIGROOT)
configh_data.set_quoted('XLOCALEDIR', XLOCALEDIR)
configh_data.set_quoted('DEFAULT_XKB_RULES', get_option('default-rules'))
configh_data.set_quoted('DEFAULT_XKB_MODEL', get_option('default-model'))
configh_data.set_quoted('DEFAULT_XKB_LAYOUT', get_option('default-layout'))
if get_option('default-variant') != ''
    configh_data.set_quoted('DEFAULT_XKB_VARIANT', get_option('default-variant'))
endif
if get_option('default-options') != ''
    configh_data.set_quoted('DEFAULT_XKB_OPTIONS', get_option('default-options'))
endif
if cc.links('int main(){if(__builtin_expect(1<0,0)){}}', name: '__builtin_expect')
    configh_data.set('HAVE___BUILTIN_EXPECT', 1)
endif
if cc.has_header_symbol('unistd.h', 'eaccess', prefix: system_ext_define)
    configh_data.set('HAVE_EACCESS', 1)
endif
if cc.has_header_symbol('unistd.h', 'euidaccess', prefix: system_ext_define)
    configh_data.set('HAVE_EUIDACCESS', 1)
endif
if cc.has_header_symbol('sys/mman.h', 'mmap')
    configh_data.set('HAVE_MMAP', 1)
endif
if cc.has_header_symbol('stdlib.h', 'mkostemp', prefix: system_ext_define)
    configh_data.set('HAVE_MKOSTEMP', 1)
endif
if cc.has_header_symbol('fcntl.h', 'posix_fallocate', prefix: system_ext_define)
    configh_data.set('HAVE_POSIX_FALLOCATE', 1)
endif
if cc.has_header_symbol('string.h', 'strndup', prefix: system_ext_define)
    configh_data.set('HAVE_STRNDUP', 1)
endif
if cc.has_header_symbol('stdio.h', 'asprintf', prefix: system_ext_define)
    configh_data.set('HAVE_ASPRINTF', 1)
elif cc.has_header_symbol('stdio.h', 'vasprintf', prefix: system_ext_define)
    configh_data.set('HAVE_VASPRINTF', 1)
endif
if cc.has_header_symbol('stdlib.h', 'secure_getenv', prefix: system_ext_define)
    configh_data.set('HAVE_SECURE_GETENV', 1)
elif cc.has_header_symbol('stdlib.h', '__secure_getenv', prefix: system_ext_define)
    configh_data.set('HAVE___SECURE_GETENV', 1)
else
    message('C library does not support secure_getenv, using getenv instead')
endif
# Silence some security & deprecation warnings on MSVC
# for some unix/C functions we use.
# https://docs.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-3-c4996?view=vs-2019
configh_data.set('_CRT_SECURE_NO_WARNINGS', 1)
configh_data.set('_CRT_NONSTDC_NO_WARNINGS', 1)
configh_data.set('_CRT_NONSTDC_NO_DEPRECATE', 1)
# Reduce unnecessary includes on MSVC.
configh_data.set('WIN32_LEAN_AND_MEAN', 1)
configure_file(output: 'config.h', configuration: configh_data)


# Supports -Wl,--version-script?
have_version_script = cc.links(
    'int main(){}',
    args: '-Wl,--version-script=' + join_paths(meson.source_root(), 'xkbcommon.map'),
    name: '-Wl,--version-script',
)


# libxkbcommon.
# Note: we use some yacc extensions, which work with either GNU bison
# (preferred) or byacc. Other yacc's may or may not work.
yacc = find_program('bison', 'win_bison', 'byacc')
yacc_gen = generator(
    yacc,
    output: ['@BASENAME@.c', '@BASENAME@.h'],
    arguments: ['@INPUT@', '--defines=@OUTPUT1@', '--output=@OUTPUT0@', '-p _xkbcommon_'],
)
libxkbcommon_sources = [
    'src/compose/parser.c',
    'src/compose/parser.h',
    'src/compose/paths.c',
    'src/compose/paths.h',
    'src/compose/state.c',
    'src/compose/table.c',
    'src/compose/table.h',
    'src/xkbcomp/action.c',
    'src/xkbcomp/action.h',
    'src/xkbcomp/ast.h',
    'src/xkbcomp/ast-build.c',
    'src/xkbcomp/ast-build.h',
    'src/xkbcomp/compat.c',
    'src/xkbcomp/expr.c',
    'src/xkbcomp/expr.h',
    'src/xkbcomp/include.c',
    'src/xkbcomp/include.h',
    'src/xkbcomp/keycodes.c',
    'src/xkbcomp/keymap.c',
    'src/xkbcomp/keymap-dump.c',
    'src/xkbcomp/keywords.c',
    yacc_gen.process('src/xkbcomp/parser.y'),
    'src/xkbcomp/parser-priv.h',
    'src/xkbcomp/rules.c',
    'src/xkbcomp/rules.h',
    'src/xkbcomp/scanner.c',
    'src/xkbcomp/symbols.c',
    'src/xkbcomp/types.c',
    'src/xkbcomp/vmod.c',
    'src/xkbcomp/vmod.h',
    'src/xkbcomp/xkbcomp.c',
    'src/xkbcomp/xkbcomp-priv.h',
    'src/atom.c',
    'src/atom.h',
    'src/context.c',
    'src/context.h',
    'src/context-priv.c',
    'src/darray.h',
    'src/keysym.c',
    'src/keysym.h',
    'src/keysym-utf.c',
    'src/ks_tables.h',
    'src/keymap.c',
    'src/keymap.h',
    'src/keymap-priv.c',
    'src/scanner-utils.h',
    'src/state.c',
    'src/text.c',
    'src/text.h',
    'src/utf8.c',
    'src/utf8.h',
    'src/utils.c',
    'src/utils.h',
]
libxkbcommon_link_args = []
if have_version_script
    libxkbcommon_link_args += '-Wl,--version-script=' + join_paths(meson.source_root(), 'xkbcommon.map')
endif
libxkbcommon = library(
    'xkbcommon',
    'xkbcommon/xkbcommon.h',
    libxkbcommon_sources,
    link_args: libxkbcommon_link_args,
    link_depends: 'xkbcommon.map',
    version: '0.0.0',
    install: true,
    include_directories: include_directories('src'),
)
install_headers(
    'xkbcommon/xkbcommon.h',
    'xkbcommon/xkbcommon-compat.h',
    'xkbcommon/xkbcommon-compose.h',
    'xkbcommon/xkbcommon-keysyms.h',
    'xkbcommon/xkbcommon-names.h',
    subdir: 'xkbcommon',
)
pkgconfig.generate(
    name: 'xkbcommon',
    filebase: 'xkbcommon',
    libraries: libxkbcommon,
    version: meson.project_version(),
    description: 'XKB API common to servers and clients',
)


# libxkbcommon-x11.
if get_option('enable-x11')
    xcb_dep = dependency('xcb', version: '>=1.10', required: false)
    xcb_xkb_dep = dependency('xcb-xkb', version: '>=1.10', required: false)
    if not xcb_dep.found() or not xcb_xkb_dep.found()
        error('''X11 support requires xcb-xkb >= 1.10 which was not found.
You can disable X11 support with -Denable-x11=false.''')
    endif

    libxkbcommon_x11_sources = [
        'src/x11/keymap.c',
        'src/x11/state.c',
        'src/x11/util.c',
        'src/x11/x11-priv.h',
        'src/context.h',
        'src/context-priv.c',
        'src/keymap.h',
        'src/keymap-priv.c',
        'src/atom.h',
        'src/atom.c',
    ]
    libxkbcommon_x11_internal = static_library(
        'xkbcommon-x11-internal',
        libxkbcommon_x11_sources,
        include_directories: include_directories('src'),
        link_with: libxkbcommon,
        dependencies: [
            xcb_dep,
            xcb_xkb_dep,
        ],
    )
    libxkbcommon_x11_link_args = []
    if have_version_script
        libxkbcommon_x11_link_args += '-Wl,--version-script=' + join_paths(meson.source_root(), 'xkbcommon-x11.map')
    endif
    libxkbcommon_x11 = library(
        'xkbcommon-x11',
        'xkbcommon/xkbcommon-x11.h',
        libxkbcommon_x11_sources,
        link_args: libxkbcommon_x11_link_args,
        link_depends: 'xkbcommon-x11.map',
        version: '0.0.0',
        install: true,
        include_directories: include_directories('src'),
        link_with: libxkbcommon,
        dependencies: [
            xcb_dep,
            xcb_xkb_dep,
        ],
    )
    install_headers(
        'xkbcommon/xkbcommon-x11.h',
        subdir: 'xkbcommon',
    )
    pkgconfig.generate(
        name: 'xkbcommon-x11',
        filebase: 'xkbcommon-x11',
        libraries: libxkbcommon_x11,
        version: meson.project_version(),
        description: 'XKB API common to servers and clients - X11 support',
        requires: ['xkbcommon'],
        requires_private: ['xcb>=1.10', 'xcb-xkb>=1.10'],
    )
endif


# Tests
test_env = environment()
test_env.set('XKB_LOG_LEVEL', 'debug')
test_env.set('XKB_LOG_VERBOSITY', '10')
test_env.set('top_srcdir', meson.source_root())
test_env.set('top_builddir', meson.build_root())

test_configh_data = configuration_data()
test_configh_data.set_quoted('TEST_XKB_CONFIG_ROOT', join_paths(meson.source_root(), 'test', 'data'))
configure_file(output: 'test-config.h', configuration: test_configh_data)

# Some tests need to use unexported symbols, so we link them against
# an internal copy of libxkbcommon with all symbols exposed.
libxkbcommon_test_internal = static_library(
    'xkbcommon-test-internal',
    'test/common.c',
    'test/test.h',
    'test/evdev-scancodes.h',
    libxkbcommon_sources,
    include_directories: include_directories('src'),
)
test_dep = declare_dependency(
    include_directories: include_directories('src'),
    link_with: libxkbcommon_test_internal,
)
if get_option('enable-x11')
    x11_test_dep = declare_dependency(
        link_with: libxkbcommon_x11_internal,
        dependencies: [
            test_dep,
            xcb_dep,
            xcb_xkb_dep,
        ],
    )
endif
test(
    'keysym',
    executable('test-keysym', 'test/keysym.c', dependencies: test_dep),
    env: test_env,
)
test(
    'keymap',
    executable('test-keymap', 'test/keymap.c', dependencies: test_dep),
    env: test_env,
)
test(
    'filecomp',
    executable('test-filecomp', 'test/filecomp.c', dependencies: test_dep),
    env: test_env,
)
# TODO: This test currently uses some functions that don't exist on Windows.
if cc.get_id() != 'msvc'
  test(
      'context',
      executable('test-context', 'test/context.c', dependencies: test_dep),
      env: test_env,
  )
endif
test(
    'rules-file',
    executable('test-rules-file', 'test/rules-file.c', dependencies: test_dep),
    env: test_env,
)
test(
    'rules-file-includes',
    executable('test-rules-file-includes', 'test/rules-file-includes.c', dependencies: test_dep),
    env: test_env,
)
test(
    'stringcomp',
    executable('test-stringcomp', 'test/stringcomp.c', dependencies: test_dep),
    env: test_env,
)
test(
    'buffercomp',
    executable('test-buffercomp', 'test/buffercomp.c', dependencies: test_dep),
    env: test_env,
)
test(
    'log',
    executable('test-log', 'test/log.c', dependencies: test_dep),
    env: test_env,
)
test(
    'atom',
    executable('test-atom', 'test/atom.c', dependencies: test_dep),
    env: test_env,
)
test(
    'utf8',
    executable('test-utf8', 'test/utf8.c', dependencies: test_dep),
    env: test_env,
)
test(
    'state',
    executable('test-state', 'test/state.c', dependencies: test_dep),
    env: test_env,
)
test(
    'keyseq',
    executable('test-keyseq', 'test/keyseq.c', dependencies: test_dep),
    env: test_env,
)
test(
    'rulescomp',
    executable('test-rulescomp', 'test/rulescomp.c', dependencies: test_dep),
    env: test_env,
)
test(
    'compose',
    executable('test-compose', 'test/compose.c', dependencies: test_dep),
    env: test_env,
)
test(
    'symbols-leak-test',
    find_program('test/symbols-leak-test.bash'),
    env: test_env,
)
if get_option('enable-x11')
    test(
        'x11',
        executable('test-x11', 'test/x11.c', dependencies: x11_test_dep),
        env: test_env,
    )
    # test/x11comp is meant to be run, but it is (temporarily?) disabled.
    # See: https://github.com/xkbcommon/libxkbcommon/issues/30
    executable('test-x11comp', 'test/x11comp.c', dependencies: x11_test_dep)
endif


# Fuzzing target programs.
executable('fuzz-keymap', 'fuzz/keymap/target.c', dependencies: test_dep)
executable('fuzz-compose', 'fuzz/compose/target.c', dependencies: test_dep)


# Demo programs.
if cc.has_header_symbol('getopt.h', 'getopt_long', prefix: '#define _GNU_SOURCE')
    executable('rmlvo-to-kccgst', 'test/rmlvo-to-kccgst.c', dependencies: test_dep)
    executable('rmlvo-to-keymap', 'test/rmlvo-to-keymap.c', dependencies: test_dep)
    executable('print-compiled-keymap', 'test/print-compiled-keymap.c', dependencies: test_dep)
endif
if cc.has_header('linux/input.h')
    executable('interactive-evdev', 'test/interactive-evdev.c', dependencies: test_dep)
endif
if get_option('enable-x11')
    executable('interactive-x11', 'test/interactive-x11.c', dependencies: x11_test_dep)
endif
if get_option('enable-wayland')
    wayland_client_dep = dependency('wayland-client', version: '>=1.2.0', required: false)
    wayland_protocols_dep = dependency('wayland-protocols', version: '>=1.12', required: false)
    wayland_scanner_dep = dependency('wayland-scanner', required: false, native: true)
    if not wayland_client_dep.found() or not wayland_protocols_dep.found() or not wayland_scanner_dep.found()
        error('''The Wayland demo programs require wayland-client >= 1.2.0, wayland-protocols >= 1.7 which were not found.
You can disable the Wayland demo programs with -Denable-wayland=false.''')
    endif

    wayland_scanner = find_program(wayland_scanner_dep.get_pkgconfig_variable('wayland_scanner'))
    wayland_scanner_code_gen = generator(
        wayland_scanner,
        output: '@BASENAME@-protocol.c',
        arguments: ['code', '@INPUT@', '@OUTPUT@'],
    )
    wayland_scanner_client_header_gen = generator(
        wayland_scanner,
        output: '@BASENAME@-client-protocol.h',
        arguments: ['client-header', '@INPUT@', '@OUTPUT@'],
    )
    wayland_protocols_datadir = wayland_protocols_dep.get_pkgconfig_variable('pkgdatadir')
    xdg_shell_xml = join_paths(wayland_protocols_datadir, 'stable/xdg-shell/xdg-shell.xml')
    xdg_shell_sources = [
        wayland_scanner_code_gen.process(xdg_shell_xml),
        wayland_scanner_client_header_gen.process(xdg_shell_xml),
    ]
    executable('interactive-wayland', 'test/interactive-wayland.c', xdg_shell_sources, dependencies: [test_dep, wayland_client_dep])
endif

# xkeyboard-config "verifier"
xkct_config = configuration_data()
xkct_config.set('MESON_BUILD_ROOT', meson.build_root())
xkct_config.set('XKB_CONFIG_ROOT', XKBCONFIGROOT)
configure_file(input: 'test/xkeyboard-config-test.py.in',
               output: 'xkeyboard-config-test',
               configuration: xkct_config,
               install: false)


# Benchmarks.
libxkbcommon_bench_internal = static_library(
    'xkbcommon-bench-internal',
    'bench/bench.c',
    'bench/bench.h',
    link_with: libxkbcommon_test_internal,
)
bench_dep = declare_dependency(
    include_directories: include_directories('src'),
    link_with: libxkbcommon_bench_internal,
)
bench_env = environment()
bench_env.set('top_srcdir', meson.source_root())
benchmark(
    'key-proc',
    executable('bench-key-proc', 'bench/key-proc.c', dependencies: bench_dep),
    env: bench_env,
)
benchmark(
    'rules',
    executable('bench-rules', 'bench/rules.c', dependencies: bench_dep),
    env: bench_env,
)
benchmark(
    'rulescomp',
    executable('bench-rulescomp', 'bench/rulescomp.c', dependencies: bench_dep),
    env: bench_env,
)
benchmark(
    'compose',
    executable('bench-compose', 'bench/compose.c', dependencies: bench_dep),
    env: bench_env,
)


# Documentation.
if get_option('enable-docs')
    doxygen = find_program('doxygen', required: false)
    if not doxygen.found()
        error('''Documentation requires doxygen which was not found.
You can disable the documentation with -Denable-docs=false.''')
    endif
    doxygen_wrapper = find_program('scripts/doxygen-wrapper')

    doxygen_input = [
        'README.md',
        'doc/doxygen-extra.css',
        'doc/quick-guide.md',
        'doc/compat.md',
        'xkbcommon/xkbcommon.h',
        'xkbcommon/xkbcommon-names.h',
        'xkbcommon/xkbcommon-x11.h',
        'xkbcommon/xkbcommon-compose.h',
    ]
    doxygen_data = configuration_data()
    doxygen_data.set('PACKAGE_NAME', meson.project_name())
    doxygen_data.set('PACKAGE_VERSION', meson.project_version())
    doxygen_data.set('INPUT', ' '.join(doxygen_input))
    doxygen_data.set('OUTPUT_DIRECTORY', meson.build_root())
    doxyfile = configure_file(
        input: 'doc/Doxyfile.in',
        output: 'Doxyfile',
        configuration: doxygen_data,
    )
    # TODO: Meson should provide this.
    docdir = join_paths(get_option('datadir'), 'doc', meson.project_name())
    custom_target(
        'doc',
        input: [doxyfile] + doxygen_input,
        output: 'html',
        command: [doxygen_wrapper, doxygen.path(), join_paths(meson.build_root(), 'Doxyfile'), meson.source_root()],
        install: true,
        install_dir: docdir,
        build_by_default: true,
    )
endif
