cmake_minimum_required (VERSION 3.1.3 FATAL_ERROR)
project (etcd-cpp-api)

# Microsoft Rest SDK
include(ExternalProject)

# Microsoft Rest SDK
set(REST_SDK_TARGET rest_sdk)
externalproject_add(${REST_SDK_TARGET}
  GIT_REPOSITORY       https://github.com/microsoft/cpprestsdk
  GIT_TAG              v2.10.15
  # don't install
  INSTALL_COMMAND      ""
  CMAKE_CACHE_ARGS     -DOPENSSL_INCLUDE_DIR:STRING=/usr/include/openssl-1.0
                       -DOPENSSL_SSL_LIBRARY:STRING=/usr/lib/openssl-1.0/libssl.so 
                       -DOPENSSL_CRYPTO_LIBRARY:STRING=/usr/lib/openssl-1.0/libcrypto.so
		       -DWERROR:BOOL=0
		       -DLDFLAGS:STRING=-L/usr/lib/openssl-1.0
)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/external/Source/rest_sdk/include)
set(CPPREST_LIB ${CMAKE_SOURCE_DIR}/external/lib)
set(CPPREST_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/rest_sdk-prefix/src/Release)
message("here we are" ${CMAKE_SOURCE_DIR})


find_library(CPPREST_LIB NAMES cpprest)
find_path(CPPREST_INCLUDE_DIR NAMES cpprest/http_client.h)

find_package(Boost REQUIRED COMPONENTS system thread locale random)

set (etcd-cpp-api_VERSION_MAJOR 0)
set (etcd-cpp-api_VERSION_MINOR 1)

enable_testing()
include_directories(SYSTEM ${CPPREST_INCLUDE_DIR} ${Boost_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic -Werror -std=c++11")

add_subdirectory(src)
add_subdirectory(tst)
