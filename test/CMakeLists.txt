# Creates test targets for the packing tests so that they can
# be executed via CMake/CTest
#
# Tests can be ran by executing the target "check", e.g., by
# writing:
#
#   make check
#
# Note: Test discovery is performed at CMake configuration time,
# so if new tests are added, they will not be tested until the
# build is reconfigured.
#
# All tests are performed with respect to the source version of
# the library, not the installed version (if any), so installation
# before running the tests is not necessary
#
# To test the installed version of the library instead, run the
# check-installed target:
#
#   make check-installed
#

include(CTest)

find_package(Python3 REQUIRED COMPONENTS Interpreter)

# Locations of the test script, the python library source,
# and the compiled library for the C++ bindings
set(TEST_PACKING "${CMAKE_CURRENT_SOURCE_DIR}/test_packing.py")
set(PYTHON_LIB_DIR ${CMAKE_SOURCE_DIR}/python)
set(BINDINGS_LIB_DIR ${CMAKE_BINARY_DIR}/src)

# Extract a list of test case names
execute_process(COMMAND ${Python3_EXECUTABLE} ${TEST_PACKING} --list-tests
                OUTPUT_VARIABLE STR_TESTS
                OUTPUT_STRIP_TRAILING_WHITESPACE
                ERROR_STRIP_TRAILING_WHITESPACE)
separate_arguments(TEST_LIST UNIX_COMMAND ${STR_TESTS})

# Create a test target for each test case.
foreach(TEST ${TEST_LIST})
  add_test(NAME ${TEST}
           COMMAND ${Python3_EXECUTABLE} ${TEST_PACKING} "-v" ${TEST}
           WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
endforeach()

# Create a single target that runs all of the tests via CTest.  We
# set the PYTHONPATH environment variable to ensure that the test
# script always loads the source version of the library, rather than
# a possibly out-of-date installed version of the library.
add_custom_target(check
  COMMAND ${CMAKE_COMMAND} -E env PYTHONPATH=${PYTHON_LIB_DIR}:${BINDINGS_LIB_DIR}:$ENV{PYTHONPATH}
  ${CMAKE_CTEST_COMMAND} --no-tests=error --output-on-failure
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_dependencies(check PackaideBindings)

# Create a target that runs all of the tests via CTest without
# loading the source libraries. This will ensure that the libraries
# are installed somewhere visible to Python
add_custom_target(check-installed
  ${CMAKE_CTEST_COMMAND} --no-tests=error --output-on-failure
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_dependencies(check PackaideBindings)
