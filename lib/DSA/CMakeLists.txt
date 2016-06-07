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
  TypeSafety.cpp
  )


set(PROJECT_NAME "LLVMDataStructure${PROJECT_TARGET_SUFFIX}")

project_add_library(${PROJECT_NAME} ${SOURCES})

set_property(TARGET ${PROJECT_NAME} PROPERTY OUTPUT_NAME "LLVMDataStructure")

# Drop the 'lib' suffix to match how the Makefile-driven version functions.
set_property(TARGET ${PROJECT_NAME} PROPERTY PREFIX "")

# intrinsics should have already been build 
# if we are using an already built LLVM project
if(NOT ${PROJECT_ENABLE_LLVM_EXTERNAL})
  add_dependencies(${PROJECT_NAME} intrinsics_gen)
endif()

# Not sure if all those are needed even for standalone execs (see tools dir).
#target_link_libraries(${PROJECT_NAME} LLVMCore)
#target_link_libraries(${PROJECT_NAME} LLVMPasses)
#target_link_libraries(${PROJECT_NAME} LLVMSupport)
#target_link_libraries(${PROJECT_NAME} LLVMAnalysis)

if(${PROJECT_ENABLE_LLVM_EXTERNAL})
  install(TARGETS ${PROJECT_NAME} 
    DESTINATION ${PROJECT_LIB_INTDIR}
    INCLUDES DESTINATION include)
endif()

