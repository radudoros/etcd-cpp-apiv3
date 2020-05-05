cmake_minimum_required (VERSION 3.1.3 FATAL_ERROR)
project (etcd-cpp-api)

find_package(Boost REQUIRED COMPONENTS system thread locale random)

set (etcd-cpp-api_VERSION_MAJOR 0)
set (etcd-cpp-api_VERSION_MINOR 1)

enable_testing()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

if(MSVC)
  
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wpedantic -Werror -std=c++17")
endif()

add_subdirectory(src)

if (${WITH_TESTS})
  add_subdirectory(tst)
endif ()  
