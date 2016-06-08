# cmake file

set(SOURCES
  ArgCast.cpp
  ArgSimplify.cpp
  DSNodeEquivs.cpp
  Devirt.cpp
  DynCount.cpp
  FuncSimplify.cpp
  FuncSpec.cpp
  GEPExprArgs.cpp
  IndCloner.cpp
  Int2PtrCmp.cpp
  LoadArgs.cpp
  MergeGEP.cpp
  SVADevirt.cpp
  SimplifyExtractValue.cpp
  SimplifyGEP.cpp
  SimplifyInsertValue.cpp
  SimplifyLoad.cpp
  StructReturnToPointer.cpp
  TypeChecks.cpp
  TypeChecksOpt.cpp)


set(PROJECT_NAME "AssistDS${PROJECT_TARGET_SUFFIX}")

project_add_library(${PROJECT_NAME} ${SOURCES})

set_property(TARGET ${PROJECT_NAME} PROPERTY OUTPUT_NAME "AssistDS")

# Drop the 'lib' suffix to match how the Makefile-driven version functions.
set_property(TARGET ${PROJECT_NAME} PROPERTY PREFIX "")

# intrinsics should have already been build 
# if we are using an already built LLVM project
if(NOT ${POOLALLOC_STANDALONE_BUILD})
  add_dependencies(${PROJECT_NAME} intrinsics_gen)
endif()

#add_dependencies(${PROJECT_NAME} LLVMDataStructure${PROJECT_TARGET_SUFFIX})

# All these are not needed if the library is to be dynamically load
# from tools like opt.
# Even for in-source LLVM builds it might be enough to denote a simple
# dependency for the required build order
target_link_libraries(${PROJECT_NAME} LLVMDataStructure${PROJECT_TARGET_SUFFIX})

if(NOT ${POOLALLOC_STANDALONE_BUILD})
  target_link_libraries(${PROJECT_NAME} LLVMCore)
  target_link_libraries(${PROJECT_NAME} LLVMPasses)
  target_link_libraries(${PROJECT_NAME} LLVMSupport)
  target_link_libraries(${PROJECT_NAME} LLVMTransformUtils)
endif()

if(${POOLALLOC_STANDALONE_BUILD})
  install(TARGETS ${PROJECT_NAME} 
    DESTINATION ${PROJECT_LIB_INTDIR}
    INCLUDES DESTINATION include)
endif()

