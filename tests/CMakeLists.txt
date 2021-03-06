
add_compile_options(-Wall -Wextra -Wunreachable-code -Wuninitialized)

add_subdirectory(
  googletest
)

include_directories(
  ${CMAKE_SOURCE_DIR}/src/ct
)

mark_as_advanced(
  BUILD_GMOCK BUILD_GTEST BUILD_SHARED_LIBS
  gmock_build_tests gtest_build_samples gtest_build_tests
  gtest_disable_pthreads gtest_force_shared_crt gtest_hide_internal_symbols
)

set_target_properties(gtest PROPERTIES FOLDER extern)
set_target_properties(gtest_main PROPERTIES FOLDER extern)
set_target_properties(gmock PROPERTIES FOLDER extern)
set_target_properties(gmock_main PROPERTIES FOLDER extern)

macro(package_add_test TESTNAME)
  # create an exectuable in which the tests will be stored
  add_executable(${TESTNAME} ${ARGN})
  # link the Google test infrastructure, mocking library, and a default main fuction to
  # the test executable.  Remove g_test_main if writing your own main function.
  target_link_libraries(${TESTNAME} gtest gmock gtest_main cherrytree_shared)
  # gtest_discover_tests replaces gtest_add_tests,
  # see https://cmake.org/cmake/help/v3.10/module/GoogleTest.html for more options to pass to it
  gtest_discover_tests(${TESTNAME}
    # set a working directory to your project root so that you can find test data via paths relative to the project root
    WORKING_DIRECTORY ${PROJECT_DIR}
    PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${PROJECT_DIR}"
  )
  set_target_properties(${TESTNAME} PROPERTIES FOLDER tests)
endmacro()

# some tests don't work in TRAVIS, so turn them off
if(DEFINED ENV{TRAVIS})
  add_definitions(-D_TRAVIS)
endif()

package_add_test(run_tests
  tests_clipboard.cpp
  tests_encoding.cpp
  tests_filesystem.cpp
  tests_main.cpp
  tests_misc_utils.cpp
  tests_read_write.cpp
  tests_tmp_n_p7zip.cpp
  tests_types.cpp
)

add_custom_command(TARGET run_tests POST_BUILD
  COMMAND ./run_tests
)
