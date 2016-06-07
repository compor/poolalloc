#include_directories(/localhome/simmon12/progs/dyncall-0.5/dyncall)
#link_directories(/localhome/simmon12/progs/dyncall-0.5/dyncall/build_out/linux_x86_gcc_release)

add_library(poolalloc_rt_sa PoolAllocator.cpp)
set_property(TARGET poolalloc_rt_sa PROPERTY COMPILE_DEFINITIONS)

#target_link_libraries(poolalloc_rt dyncall_s)
