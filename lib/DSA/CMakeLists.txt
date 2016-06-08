# cmake file

set(SOURCES
  AddressTakenAnalysis.cpp
  AllocatorIdentification.cpp
  Basic.cpp
  BottomUpClosure.cpp
  CallTargets.cpp
  CompleteBottomUp.cpp
  DSCallGraph.cpp
  DSGraph.cpp
  DSTest.cpp
  DataStructure.cpp
  DataStructureStats.cpp
  EntryPointAnalysis.cpp
  EquivClassGraphs.cpp
  GraphChecker.cpp
  Local.cpp
  Printer.cpp
  SanityCheck.cpp
  StdLibPass.cpp
  TopDownClosure.cpp
  TypeSafety.cpp)


set(PROJECT_NAME "LLVMDataStructure${PROJECT_TARGET_SUFFIX}")

project_add_library(${PROJECT_NAME} ${SOURCES})

set_property(TARGET ${PROJECT_NAME} PROPERTY OUTPUT_NAME "LLVMDataStructure")

# Drop the 'lib' suffix to match how the Makefile-driven version functions.
set_property(TARGET ${PROJECT_NAME} PROPERTY PREFIX "")

# intrinsics should have already been build 
# if we are using an already built LLVM project
if(NOT ${POOLALLOC_STANDALONE_BUILD})
  add_dependencies(${PROJECT_NAME} intrinsics_gen)
endif()

# All these are not needed if the library is to be dynamically load
# from tools like opt.
# Even for in-source LLVM builds it might be enough to denote a simple
# dependency for the required build order
if(NOT ${POOLALLOC_STANDALONE_BUILD})
  target_link_libraries(${PROJECT_NAME} LLVMCore)
  target_link_libraries(${PROJECT_NAME} LLVMPasses)
  target_link_libraries(${PROJECT_NAME} LLVMSupport)
  target_link_libraries(${PROJECT_NAME} LLVMAnalysis)
endif()

if(${POOLALLOC_STANDALONE_BUILD})
  install(TARGETS ${PROJECT_NAME} 
    DESTINATION ${PROJECT_LIB_INTDIR}
    INCLUDES DESTINATION include)
endif()

