set(SOURCES
  AccessTrace.cpp
  AllHeapNodesHeuristic.cpp
  AllNodesHeuristic.cpp
  Heuristic.cpp
  PAMultipleGlobalPool.cpp
  PASimple.cpp
  PointerCompress.cpp
  PoolAllocate.cpp
  PoolOptimize.cpp
  RunTimeAssociate.cpp
  TransformFunctionBody.cpp
)


set(PROJECT_NAME "poolalloc${PROJECT_TARGET_SUFFIX}")

project_add_library(${PROJECT_NAME} ${SOURCES})

set_property(TARGET ${PROJECT_NAME} PROPERTY OUTPUT_NAME "poolalloc")

# Drop the 'lib' suffix to match how the Makefile-driven version functions.
set_property(TARGET ${PROJECT_NAME} PROPERTY PREFIX "")

# intrinsics should have already been build 
# if we are using an already built LLVM project
if(NOT ${PROJECT_ENABLE_LLVM_EXTERNAL})
  add_dependencies(${PROJECT_NAME} intrinsics_gen)
endif()

# Not sure if all those are needed even for standalone execs (see tools dir).
target_link_libraries(${PROJECT_NAME} LLVMDataStructure)
#target_link_libraries(${PROJECT_NAME} LLVMTransformUtils)

if(${PROJECT_ENABLE_LLVM_EXTERNAL})
  install(TARGETS ${PROJECT_NAME} 
    DESTINATION ${PROJECT_LIB_INTDIR}
    INCLUDES DESTINATION include)
endif()

