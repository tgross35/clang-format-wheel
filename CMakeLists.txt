project(clang-format-wheel)
cmake_minimum_required(VERSION 3.16)

# Include the "single source of truth" for the clang-format version
include(clang-format_version.cmake)
string(REPLACE "-" "" CLANG_FORMAT_VERSION_SHORT "${CLANG_FORMAT_VERSION}")

# Define a build rule clang-format
set(LLVM_DOWNLOAD_URL "https://github.com/llvm/llvm-project/releases/download/llvmorg-${CLANG_FORMAT_VERSION}/llvm-project-${CLANG_FORMAT_VERSION_SHORT}.src.tar.xz")
include(ExternalProject)
ExternalProject_add(build-clang-format
  URL "${LLVM_DOWNLOAD_URL}"
  SOURCE_SUBDIR llvm
  SOURCE_DIR ${CMAKE_BINARY_DIR}/llvm-project
  BINARY_DIR ${CMAKE_BINARY_DIR}/llvm
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
  USES_TERMINAL_DOWNLOAD 1
  USES_TERMINAL_CONFIGURE 1
  USES_TERMINAL_BUILD 1
  CMAKE_ARGS -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DLLVM_ENABLE_PROJECTS=clang
  BUILD_COMMAND ${CMAKE_COMMAND} --build . --target clang-format
)
set(clang-format-executable ${CMAKE_BINARY_DIR}/llvm/bin/clang-format${CMAKE_EXECUTABLE_SUFFIX})

# Reduce the size of the executable by executing strip if it is present on the system
find_program(STRIP_EXECUTABLE strip)
if(STRIP_EXECUTABLE)
  add_custom_target(
    strip-clang-format
    ALL
    COMMAND ${STRIP_EXECUTABLE} ${clang-format-executable}
    COMMENT "Stripping clang-format executable for size reduction"
  )
  add_dependencies(strip-clang-format build-clang-format)
endif()

# Define an installation rule that copies the executable to our Python package
install(
  PROGRAMS
    ${clang-format-executable}
    ${CMAKE_BINARY_DIR}/llvm-project/clang/tools/clang-format/clang-format-diff.py
    ${CMAKE_BINARY_DIR}/llvm-project/clang/tools/clang-format/git-clang-format
  DESTINATION clang_format/data/bin
)
