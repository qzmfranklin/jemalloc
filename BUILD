# Description:
# jemalloc - a general-purpose scalable concurrent malloc implementation

licenses(['notice'])

load('//tools/bazel_rules:template.bzl', 'template_rule')

# Works with jemalloc 5.0.1.

cc_library(
    name = 'jemalloc',
    visibility = [
        '//visibility:public'
    ],
    srcs = glob([
        'src/*.c',
        'src/jemalloc_cpp.cpp',
    ], exclude = [
        # This source file is for zones on Darwin (OS X).
        'src/zone.c',
    ]),
    hdrs = [
        ':generated_headers',
    ],
    textual_hdrs = glob([
        'include/jemalloc/internal/*.h',
    ]),
    copts = [
        # Suppress known warnings.
        '-Wno-ignored-attributes',

        '-D_GNU_SOURCE',
        '-D_REENTRANT',
        '-O3',
        '-funroll-loops',
        '-fvisibility=hidden',
        '-pipe',

        '-I$(GENDIR)',
    ],
    includes = [
        'include',
    ],
    linkopts = [
        '-ldl',
        '-lm',
        '-lpthread',
    ],
)

genrule(
    name = 'generated_headers',
    tools = [
        'include/jemalloc/internal/private_symbols.sh',
    ],
    srcs = glob([
        '.autom4te.cfg',
        'configure.ac',
        'm4/ax_cxx_compile_stdcxx.m4',
        '**/*.in',
        'build-aux/config.guess',
        'build-aux/config.sub',
        'build-aux/install-sh',
        'include/jemalloc/*.sh',
        'include/jemalloc/internal/*.sh',
        'include/jemalloc/internal/*.h',
    ]) + [
        'src/%s.c' % f for f in PRE_COMPILE
    ],
    outs = GENERATED_HEADERS,
    cmd = ' && '.join([
        # Generate everything but
        # include/jemalloc/internal/private_namespace.h
        'cd %s' % PACKAGE_NAME,
        'autoconf',
        './configure &> /dev/null',
        'cd -',
    ] + [
        # Generate the .sym files and the corresponding .sym.o files.
        r'''gcc -o {target}.sym.o -c $(location src/{target}.c) \
                -std=gnu11 -pipe -g3 -fvisibility=hidden -O3 \
                -D_GNU_SOURCE -D_REENTRANT -DJEMALLOC_NO_PRIVATE_NAMESPACE \
                -I{pkgname}/include \
                && nm -a {target}.sym.o | gawk -f \
                {pkgname}/include/jemalloc/internal/private_symbols.awk \
                    > {target}.sym'''.format(target = f,
                                             pkgname = PACKAGE_NAME) \
        for f in PRE_COMPILE
    ] + [
        ## Generate private_namespace.h using the generated .sym files.
        r'''$(location include/jemalloc/internal/private_namespace.sh) %s \
                > %s/include/jemalloc/internal/private_namespace.h''' %
                (' '.join(['%s.sym' % f for f in PRE_COMPILE]), PACKAGE_NAME)
    ] + [
        # Copy the generated header files to the proper locations.
        'mv %s/%s $(location %s)' % (PACKAGE_NAME, f, f) \
                for f in GENERATED_HEADERS
    ]),
)

GENERATED_HEADERS = [
    'include/jemalloc/internal/jemalloc_internal_defs.h',
    'include/jemalloc/internal/jemalloc_preamble.h',
    'include/jemalloc/internal/private_namespace.h',
    'include/jemalloc/internal/public_namespace.h',
    'include/jemalloc/internal/public_symbols.txt',
    'include/jemalloc/internal/public_unnamespace.h',
    'include/jemalloc/internal/size_classes.h',
    'include/jemalloc/jemalloc.h',
    'include/jemalloc/jemalloc_defs.h',
    'include/jemalloc/jemalloc_macros.h',
    'include/jemalloc/jemalloc_mangle.h',
    'include/jemalloc/jemalloc_mangle_jet.h',
    'include/jemalloc/jemalloc_protos.h',
    'include/jemalloc/jemalloc_protos_jet.h',
    'include/jemalloc/jemalloc_rename.h',
    'include/jemalloc/jemalloc_typedefs.h',
]

PRE_COMPILE = [
    'arena',
    'background_thread',
    'base',
    'bitmap',
    'ckh',
    'ctl',
    'extent',
    'extent_dss',
    'extent_mmap',
    'hash',
    'hooks',
    'jemalloc',
    'large',
    'malloc_io',
    'mutex',
    'mutex_pool',
    'nstime',
    'pages',
    'prng',
    'prof',
    'rtree',
    'spin',
    'stats',
    'sz',
    'tcache',
    'ticker',
    'tsd',
    'witness',
]
