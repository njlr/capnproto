
# kj ===========================================================================

set(kj_sources_lite
  array.c++
  common.c++
  debug.c++
  exception.c++
  io.c++
  memory.c++
  mutex.c++
  string.c++
  thread.c++
  main.c++
  arena.c++
)
set(kj_sources_heavy
  units.c++
  refcount.c++
  string-tree.c++
)
set(kj-parse_sources
  parse/char.c++
)
set(kj_sources ${kj_sources_lite})
if(NOT CAPNP_LITE)
  list(APPEND kj_sources ${kj_sources_heavy})
endif()
if(CAPNP_BUILD_TOOLS)
  list(APPEND kj_sources ${kj-parse_sources})
endif()

set(kj_headers
  common.h
  units.h
  memory.h
  refcount.h
  array.h
  vector.h
  string.h
  string-tree.h
  exception.h
  debug.h
  arena.h
  miniposix.h
  io.h
  tuple.h
  one-of.h
  function.h
  mutex.h
  thread.h
  threadlocal.h
  main.h
  windows-sanity.h
)
set(kj-parse_headers
  parse/common.h
  parse/char.h
)
set(kj-std_headers
  std/iostream.h
)
add_library(kj ${kj_sources})
add_library(CapnProto::kj ALIAS kj)
if(UNIX AND NOT ANDROID)
  target_link_libraries(kj PUBLIC pthread)
endif()
#make sure the lite flag propagates to all users (internal + external) of this library
target_compile_definitions(kj PUBLIC ${CAPNP_LITE_FLAG})
#make sure external consumers don't need to manually set the include dirs
target_include_directories(kj INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
  $<INSTALL_INTERFACE:include>
)
install(TARGETS kj ${INSTALL_TARGETS_DEFAULT_ARGS})
install(FILES ${kj_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj")
install(FILES ${kj-parse_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj/parse")
install(FILES ${kj-std_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj/std")

set(kj-test_sources
  test.c++
)
set(kj-test_headers
  test.h
)
set(kj-test-compat_headers
  compat/gtest.h
)
add_library(kj-test ${kj-test_sources})
add_library(CapnProto::kj-test ALIAS kj-test)
target_link_libraries(kj-test kj)
install(TARGETS kj-test ${INSTALL_TARGETS_DEFAULT_ARGS})
install(FILES ${kj-test_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj")
install(FILES ${kj-test-compat_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj/compat")

set(kj-async_sources
  async.c++
  async-unix.c++
  async-win32.c++
  async-io-win32.c++
  async-io.c++
  async-io-unix.c++
  time.c++
)
set(kj-async_headers
  async-prelude.h
  async.h
  async-inl.h
  async-unix.h
  async-win32.h
  async-io.h
  time.h
)
if(NOT CAPNP_LITE)
  add_library(kj-async ${kj-async_sources})
  add_library(CapnProto::kj-async ALIAS kj-async)
  target_link_libraries(kj-async PUBLIC kj)
  if(UNIX)
    # external clients of this library need to link to pthreads
    target_compile_options(kj-async INTERFACE "-pthread")
  elseif(WIN32)
    target_link_libraries(kj-async PUBLIC ws2_32)
  endif()
  install(TARGETS kj-async ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ${kj-async_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/kj")
endif()

# Tests ========================================================================

if(BUILD_TESTING)
  add_executable(kj-tests
    common-test.c++
    memory-test.c++
    array-test.c++
    string-test.c++
    exception-test.c++
    debug-test.c++
    io-test.c++
    mutex-test.c++
    threadlocal-test.c++
    test-test.c++
    std/iostream-test.c++
  )
  # TODO: Link with librt on Solaris for sched_yield
  target_link_libraries(kj-tests kj-test kj)
  add_dependencies(check kj-tests)
  add_test(NAME kj-tests-run COMMAND kj-tests)

  if(NOT CAPNP_LITE)
    add_executable(kj-heavy-tests
      async-test.c++
      async-unix-test.c++
      async-win32-test.c++
      async-io-test.c++
      refcount-test.c++
      string-tree-test.c++
      arena-test.c++
      units-test.c++
      tuple-test.c++
      one-of-test.c++
      function-test.c++
      threadlocal-pthread-test.c++
    )
    target_link_libraries(kj-heavy-tests kj-async kj-test kj)
    add_dependencies(check kj-heavy-tests)
    add_test(NAME kj-heavy-tests-run COMMAND kj-heavy-tests)

    if(CAPNP_BUILD_TOOLS)
      add_executable(kj-parse-tests
        parse/common-test.c++
        parse/char-test.c++
      )
      target_link_libraries(kj-parse-tests kj-test kj)
      add_dependencies(check kj-parse-tests)
      add_test(NAME kj-parse-tests-run COMMAND kj-parse-tests)
    endif()  # CAPNP_BUILD_TOOLS
  endif()  # NOT CAPNP_LITE
endif()  # BUILD_TESTING
