# Installs the Python library

find_package(Python3 REQUIRED COMPONENTS Interpreter)

# Add an installation target for the Python library
set(SETUP_PY "${CMAKE_CURRENT_SOURCE_DIR}/setup.py")
install(CODE "execute_process(
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMAND ${Python3_EXECUTABLE} ${SETUP_PY} install)
")
