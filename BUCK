include_defs('//BUCKAROO_DEPS')

cxx_library(
  name = 'capnproto',
  header_namespace = 'capnp',
  exported_headers = subdir_glob([
    ('c++/src/capnp', '**/*.h'),
  ], 
  excludes = glob([
    'c++/src/capnp/**/test*.h',
  ])),
  srcs = glob([
    'c++/src/capnp/**/*.c++',
  ], 
  excludes = glob([
    'c++/src/capnp/benchmark/**/*.c++',
    'c++/src/capnp/**/*test*.c++',
  ])),
  compiler_flags = [
    '-std=c++14',
    '-Wno-inconsistent-missing-override',
  ],
  visibility = [
    'PUBLIC',
  ],
  deps = BUCKAROO_DEPS + [
    ':kj',
  ],
)

cxx_library(
  name = 'kj',
  header_namespace = 'kj',
  exported_headers = subdir_glob([
    ('c++/src/kj', '**/*.h'),
  ]),
  srcs = glob([
    'c++/src/kj/**/*.c++',
  ], 
  excludes = glob([
    'c++/src/kj/compat/**/*.c++',
    'c++/src/kj/**/*-test.c++',
  ])),
  compiler_flags = [
    '-std=c++14',
  ],
)
