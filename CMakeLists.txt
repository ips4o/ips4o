cmake_minimum_required(VERSION 3.10)

project(ips4o)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/)
include(default_build_type)

SET(CMAKE_CXX_STANDARD 17)

add_library(ips4o INTERFACE)
target_include_directories(ips4o INTERFACE include/)

option(IPS4O_OPTIMIZE_FOR_NATIVE "Build with -march=native" ON)
if (IPS4O_OPTIMIZE_FOR_NATIVE)
  include(CheckCXXCompilerFlag)
  CHECK_CXX_COMPILER_FLAG("-march=native" COMPILER_SUPPORTS_MARCH_NATIVE)
  if(COMPILER_SUPPORTS_MARCH_NATIVE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
    message(STATUS "IPS4O_OPTIMIZE_FOR_NATIVE: ${IPS4O_OPTIMIZE_FOR_NATIVE}")
  else()
    message(SEND_ERROR "IPS4O_OPTIMIZE_FOR_NATIVE: ${IPS4O_OPTIMIZE_FOR_NATIVE}")
  endif()
endif()

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  option(IPS4O_USE_MCX16 "Enable 128-bit atomic instructions explicitly with the compile option -mcx16 (only required if not optimized for native)" OFF)
  if (IPS4O_USE_MCX16)
    target_compile_options(ips4o INTERFACE "-mcx16")
  endif()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  option(IPS4O_USE_MCX16 "Enable 128-bit atomic instructions explicitly with the compile option -mcx16 (only required if not optimized for native)" OFF)
  if (IPS4O_USE_MCX16)
    target_compile_options(ips4o INTERFACE "-mcx16")
  endif()
endif()

option(IPS4O_DISABLE_PARALLEL "Disable parallel code of IPS4o" OFF)
if (NOT IPS4O_DISABLE_PARALLEL)

  set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
  set(THREADS_PREFER_PTHREAD_FLAG TRUE)
  find_package (Threads REQUIRED)
  target_link_libraries(ips4o INTERFACE Threads::Threads)

  find_package(TBB REQUIRED)
  target_link_libraries(ips4o INTERFACE TBB::tbb)

  target_link_libraries(ips4o INTERFACE atomic)
  
endif()
message(STATUS "Parallel support of IPS4o disabled: ${IPS4O_DISABLE_PARALLEL}")

option(IPS4O_USE_OPENMP "Enable OpenMP threads" OFF)
if (IPS4O_USE_OPENMP)
  if (NOT IPS4O_DISABLE_PARALLEL)
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
      target_link_libraries(ips4o INTERFACE OpenMP::OpenMP_CXX)
    endif()
  endif()
endif()

add_executable(ips4o_example src/example.cpp)
target_link_libraries(ips4o_example PRIVATE ips4o)
