# cmake file

cmake_minimum_required(VERSION 3.2)
set(PROJECT_NAME "poolalloc")

# Standalone build or part of LLVM?
set(POOLALLOC_STANDALONE_BUILD FALSE)

if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_SOURCE_DIR}")
  project(${PROJECT_NAME} C CXX)
  set(POOLALLOC_STANDALONE_BUILD TRUE)
endif()

message(STATUS "Configuring ${PROJECT_NAME} as external LLVM project:\
 ${POOLALLOC_STANDALONE_BUILD}")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

if(${POOLALLOC_STANDALONE_BUILD})
  # find_package in CONFIG mode will lookup and include LLVMConfig.cmake
  find_package(LLVM REQUIRED CONFIG)
endif()

set(LLVM_RUNTIME_OUTPUT_INTDIR "${CMAKE_BINARY_DIR}/${CMAKE_CFG_INT_DIR}/bin")
set(LLVM_LIBRARY_OUTPUT_INTDIR "${CMAKE_BINARY_DIR}/${CMAKE_CFG_INT_DIR}/lib")


#

# These are required if functions are used:
# - add_llvm_loadable_module()
# - add_llvm_library
if(${POOLALLOC_STANDALONE_BUILD})
  list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
else()
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_BINARY_DIR}/share/llvm/cmake/")
endif()

include(AddLLVM)

if(NOT ${LLVM_ENABLE_RTTI})
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
endif()

message(STATUS "Configuring ${PROJECT_NAME} RTTI support: ${LLVM_ENABLE_RTTI}")

if(NOT ${LLVM_ENABLE_EH})
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")
endif()

message(STATUS "Configuring ${PROJECT_NAME} EH support: ${LLVM_ENABLE_EH}")


function(project_add_library fname)
if(${POOLALLOC_STANDALONE_BUILD})
  add_library(${fname} ${ARGN})
else()
  add_llvm_library(${fname} ${ARGN})
endif()
endfunction()


if(${POOLALLOC_STANDALONE_BUILD})
  # This will be set by LLVM if it is built as in-source LLVM project.
  set(BUILD_SHARED_LIBS ON)

  if(WIN32)
	set(BUILD_SHARED_LIBS OFF)
  endif()

  # The suffix allows to differentiate between standalone
  # and in-source LLVM builds, but can be changed on demand.
  if(NOT DEFINED PROJECT_TARGET_SUFFIX)
	set(PROJECT_TARGET_SUFFIX "_ext")
  endif()

  set(PROJECT_LIB_INTDIR "lib")
endif()

message(STATUS "Configuring ${PROJECT_NAME} with target suffix:\
  \"${PROJECT_TARGET_SUFFIX}\"")

# 

include_directories(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include
                    ${CMAKE_CURRENT_BINARY_DIR}/include)

include(CheckIncludeFiles)
check_include_files("fcnl.h" HAVE_FCNTL_H)
check_include_files("sys/mman.h" HAVE_SYS_MMAN_H)
check_include_files("sys/stat.h" HAVE_SYS_STAT_H)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/include/poolalloc/Config/config.h.cmake
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

if(${POOLALLOC_STANDALONE_BUILD})
  install(DIRECTORY include
	DESTINATION .
	PATTERN "Config" EXCLUDE)

  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include
	DESTINATION .)
endif()

