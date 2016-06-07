cmake_minimum_required(VERSION 3.2)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

find_package(LLVM REQUIRED CONFIG)

set(LLVM_RUNTIME_OUTPUT_INTDIR "${CMAKE_BINARY_DIR}/${CMAKE_CFG_INT_DIR}/bin")
set(LLVM_LIBRARY_OUTPUT_INTDIR "${CMAKE_BINARY_DIR}/${CMAKE_CFG_INT_DIR}/lib")


# Determine if the project is build as standalone
# outside of the LLVM source tree

set(PROJECT_ENABLE_LLVM_EXTERNAL OFF)

# The determination of whether the project is built as standalone or
# as in-source LLVM is based on the existence of LLVM's version variable.
# We could get paranoid and add directory check or even
# going through the LLVM external projects variables. 

if(NOT DEFINED ${LLVM_PACKAGE_VERSION})
  message(STATUS "configuring as an external standalone project")
  set(PROJECT_ENABLE_LLVM_EXTERNAL ON)
endif()


#

# These are required if functions are used:
# - add_llvm_loadable_module()
# - add_llvm_library
if(NOT ${PROJECT_ENABLE_LLVM_EXTERNAL})
  list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
  include(AddLLVM)
endif()


function(project_add_library fname)
if(${PROJECT_ENABLE_LLVM_EXTERNAL})
  add_library(${fname} ${ARGN})
else()
  add_llvm_library(${fname} ${ARGN})
endif()
endfunction()


if(${PROJECT_ENABLE_LLVM_EXTERNAL})
  # This will be set by LLVM if it is built as in-source LLVM project.
  set(BUILD_SHARED_LIBS ON)

  if(WIN32)
	set(BUILD_SHARED_LIBS OFF)
  endif()

  # The suffix allows to differentiate between standalone
  # and in-source LLVM builds, but can be changed on demand.
  if(NOT DEFINED ${PROJECT_TARGET_SUFFIX})
	set(PROJECT_TARGET_SUFFIX "_ext")
  endif()

  set(PROJECT_LIB_INTDIR "lib")
endif()


# 

include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include
                    ${CMAKE_CURRENT_BINARY_DIR}/include)

include(CheckIncludeFiles)
check_include_files("fcnl.h" HAVE_FCNTL_H)
check_include_files("sys/mman.h" HAVE_SYS_MMAN_H)
check_include_files("sys/stat.h" HAVE_SYS_STAT_H)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/poolalloc/Config/config.h.cmake
			   ${CMAKE_CURRENT_BINARY_DIR}/include/poolalloc/Config/config.h)

add_subdirectory(lib)
add_subdirectory(runtime)

if(NOT ${PROJECT_ENABLE_TOOLS})
  add_subdirectory(tools)
endif()

if(NOT ${PROJECT_ENABLE_TEST})
  add_subdirectory(test)
endif()


#

# The header file produced by configure should be installed
# only if it affects the public interface (e.g. included by a header file).
# In this case, it currently does affect it.

if(${PROJECT_ENABLE_LLVM_EXTERNAL})
  install(DIRECTORY include
	DESTINATION .
	PATTERN "Config" EXCLUDE)

  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include
	DESTINATION .)
endif()

