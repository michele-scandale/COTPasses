cmake_minimum_required(VERSION 2.8.12.2)

if(POLICY CMP0022)
  cmake_policy(SET CMP0022 NEW) # automatic when 2.8.12 is required
endif()

if (POLICY CMP0051)
  # CMake 3.1 and higher include generator expressions of the form
  # $<TARGETLIB:obj> in the SOURCES property.  These need to be
  # stripped everywhere that access the SOURCES property, so we just
  # defer to the OLD behavior of not including generator expressions
  # in the output for now.
  cmake_policy(SET CMP0051 OLD)
endif()

if(CMAKE_VERSION VERSION_LESS 3.1.20141117)
  set(cmake_3_2_USES_TERMINAL)
else()
  set(cmake_3_2_USES_TERMINAL USES_TERMINAL)
endif()

project(COTPasses)

set(PATH_TO_LLVM_SOURCE "" CACHE PATH
    "Path to LLVM source code. Not necessary if using an installed LLVM.")
set(LLVM_BINARY_DIR "" CACHE PATH "llvm build")
set(LLVM_MAIN_BUILD_DIR ${LLVM_BINARY_DIR})
set(PATH_TO_LLVM_BUILD ${LLVM_BINARY_DIR})

if(PATH_TO_LLVM_SOURCE)
  if(NOT EXISTS "${PATH_TO_LLVM_SOURCE}/cmake/config-ix.cmake")
    message(FATAL_ERROR "Please set PATH_TO_LLVM_SOURCE to the root "
            "directory of LLVM source code.")
  else()
    get_filename_component(ABSOLUTE LLVM_MAIN_SRC_DIR ${PATH_TO_LLVM_SOURCE})
    list(APPEND CMAKE_MODULE_PATH "${LLVM_MAIN_SRC_DIR}/cmake/modules")
  endif()
endif()

list(APPEND CMAKE_MODULE_PATH "${PATH_TO_LLVM_BUILD}/share/llvm/cmake")

#get_filename_component(ABSOLUTE LLVM_MAIN_BUILD_DIR ${PATH_TO_LLVM_BUILD})
#get_filename_component(ABSOLUTE LLVM_MAIN_BUILD_DIR ${LLVM_BINARY_DIR})
#set(LLVM_MAIN_BUILD_DIR "/home/keru/Documenti/llvm-work/build" CACHE PATH "")

include(AddLLVM)
include(HandleLLVMOptions)

set(LLVM_MAIN_INCLUDE_DIR "${LLVM_MAIN_SRC_DIR}/include")
set(LLVM_SOURCE_DIR "${LLVM_MAIN_SRC_DIR}")
set(LLVM_LIBRARY_DIR "${LLVM_MAIN_BUILD_DIR}/lib")

set(CMAKE_INCLUDE_CURRENT_DIR ON)
include_directories("${LLVM_MAIN_BUILD_DIR}/include"
                    "${LLVM_MAIN_INCLUDE_DIR}")
link_directories("${LLVM_LIBRARY_DIR}")

if(EXISTS "${LLVM_MAIN_BUILD_DIR}/bin/llvm-config${CMAKE_EXECUTABLE_SUFFIX}")
  set(PATH_TO_LLVM_CONFIG "${LLVM_MAIN_BUILD_DIR}/bin/llvm-config${CMAKE_EXECUTABLE_SUFFIX}")
else()
  message(FATAL_ERROR "Please set PATH_TO_LLVM_BUILD to a directory containing a LLVM build.")
endif()

exec_program("${PATH_TO_LLVM_CONFIG} --obj-root" OUTPUT_VARIABLE LLVM_BINARY_DIR)
exec_program("${PATH_TO_LLVM_CONFIG} --bindir" OUTPUT_VARIABLE LLVM_TOOLS_DIR)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(COTPASSES_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(COTPASSES_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(COTPASSES_SHLIB_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
  message(FATAL_ERROR "In-source builds are not allowed. CMake would overwrite "
"the makefiles distributed with LLVM. Please create a directory and run cmake "
"from there, passing the path to this source directory as the last argument. "
"This process created the file `CMakeCache.txt' and the directory "
"`CMakeFiles'. Please delete them.")
endif()

# gcc and clang require the -std=c++0x or -std=c++11 flag.
if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU|Clang" AND
    NOT ("${CMAKE_CXX_FLAGS}" MATCHES ".*-std=(c|gnu)\\+\\+(0x|11|1y).*"))
  message(FATAL_ERROR
    "COTPasses requires c++11. Clang and gcc require -std=c++0x or -std=c++11 "
    "to enter this mode. Please set CMAKE_CXX_FLAGS accordingly.")
endif()

include_directories(BEFORE
  ${CMAKE_CURRENT_BINARY_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include
)

add_subdirectory(lib)
add_subdirectory(tools)
add_subdirectory(test)
