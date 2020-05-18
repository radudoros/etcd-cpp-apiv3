find_package(Boost REQUIRED COMPONENTS system thread locale random)

find_package(gRPC)
  set(GRPCPP_LIBRARIES gRPC::grpc++)
if(NOT ${gRPC_FOUND})
  message("Falling back to finding gRPC with pkg-config")
  find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin)
  if (NOT GRPC_CPP_PLUGIN)
    message(FATAL_ERROR "grpc_cpp_plugin not found!")
  endif()
  find_package(PkgConfig REQUIRED)
  pkg_search_module(GRPC REQUIRED grpc)
  pkg_search_module(GRPCPP REQUIRED grpc++)
  list(APPEND GRPCPP_INCLUDE_DIRS ${GRPC_INCLUDE_DIRS})
  list(APPEND GRPCPP_LIBRARIES ${GRPC_LIBRARIES})
endif()

find_package(OpenSSL 1.0.0 REQUIRED)
find_package(Protobuf REQUIRED)

find_program(PROTOBUF_PROTOC protoc)
find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin)

add_custom_command(
  OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/../proto/rpc.grpc.pb.cc" "${CMAKE_CURRENT_SOURCE_DIR}/../proto/rpc.grpc.pb.h"
  COMMAND ${PROTOBUF_PROTOC}
  ARGS -I "${CMAKE_CURRENT_SOURCE_DIR}/../proto"
    --grpc_out="${CMAKE_CURRENT_SOURCE_DIR}/../proto"
    --plugin=protoc-gen-grpc="${GRPC_CPP_PLUGIN}"
    "${CMAKE_CURRENT_SOURCE_DIR}/../proto/rpc.proto"
  )

add_custom_command(
  OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/../proto/kv.pb.cc" "${CMAKE_CURRENT_SOURCE_DIR}/../proto/kv.pb.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/../proto/auth.pb.cc" "${CMAKE_CURRENT_SOURCE_DIR}/../proto/auth.pb.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/../proto/rpc.pb.cc" "${CMAKE_CURRENT_SOURCE_DIR}/../proto/rpc.pb.h"
  COMMAND ${PROTOBUF_PROTOC}
  ARGS -I "${CMAKE_CURRENT_SOURCE_DIR}/../proto"
    --cpp_out="${CMAKE_CURRENT_SOURCE_DIR}/../proto"
    "${CMAKE_CURRENT_SOURCE_DIR}/../proto/*.proto"
  )

add_library(etcd-cpp-api SHARED 
  ../proto/kv.pb.cc
  ../proto/auth.pb.cc
  ../proto/rpc.pb.cc
  ../proto/rpc.grpc.pb.cc
  ../v3/src/AsyncTxnResponse.cpp
  ../v3/src/AsyncRangeResponse.cpp
  ../v3/src/Transaction.cpp
  ../v3/src/action_constants.cpp
  ../v3/src/AsyncSetAction.cpp
  ../v3/src/AsyncCompareAndSwapAction.cpp
  ../v3/src/AsyncUpdateAction.cpp
  ../v3/src/AsyncGetAction.cpp
  ../v3/src/AsyncDeleteAction.cpp
  ../v3/src/AsyncCompareAndDeleteAction.cpp
  ../v3/src/Action.cpp
  ../v3/src/AsyncWatchAction.cpp
  ../v3/src/V3Response.cpp
  ../v3/src/AsyncDeleteRangeResponse.cpp
  ../v3/src/AsyncWatchResponse.cpp
  ../v3/src/AsyncLeaseGrantResponse.cpp
  ../v3/src/AsyncLeaseGrantAction.cpp
  ../v3/src/KeyValue.cpp
  Client.cpp
  Response.cpp
  SyncClient.cpp
  Value.cpp
  Watcher.cpp)

# TODO: find a way to build external_projects with CXX_STANDARD 17
# set_property(TARGET etcd-cpp-api PROPERTY CXX_STANDARD 17)

if(MSVC)
  set_target_properties(etcd-cpp-api PROPERTIES LINK_FLAGS "/ignore:4251")
endif()

target_link_libraries(etcd-cpp-api ${Boost_SYSTEM_LIBRARY} ${CPPREST_LIB} ${GRPCPP_LIBRARIES} ${OPENSSL_LIBRARIES} ${Protobuf_LIBRARIES})
target_include_directories(etcd-cpp-api PRIVATE ${CPPREST_INCLUDE_DIR} ${OPENSSL_INCLUDE_DIR} ${Protobuf_INCLUDE_DIRS})

install (TARGETS etcd-cpp-api DESTINATION lib)
install (FILES ../etcd/Client.hpp
               ../etcd/SyncClient.hpp
               ../etcd/Response.hpp
               ../etcd/Value.hpp
               ../etcd/Watcher.hpp
               DESTINATION include/etcd)

set(V3_DIR ../v3/include)
set(V3_INCLUDES
  ${V3_DIR}/action_constants.hpp
  ${V3_DIR}/Action.hpp
  ${V3_DIR}/AsyncCompareAndDeleteAction.hpp
  ${V3_DIR}/AsyncCompareAndSwapAction.hpp
  ${V3_DIR}/AsyncDeleteAction.hpp
  ${V3_DIR}/AsyncDeleteRangeResponse.hpp
  ${V3_DIR}/AsyncGetAction.hpp
  ${V3_DIR}/AsyncLeaseGrantAction.hpp
  ${V3_DIR}/AsyncLeaseGrantResponse.hpp
  ${V3_DIR}/AsyncRangeResponse.hpp
  ${V3_DIR}/AsyncSetAction.hpp
  ${V3_DIR}/AsyncTxnResponse.hpp
  ${V3_DIR}/AsyncUpdateAction.hpp
  ${V3_DIR}/AsyncWatchAction.hpp
  ${V3_DIR}/AsyncWatchResponse.hpp
  ${V3_DIR}/KeyValue.hpp
  ${V3_DIR}/Transaction.hpp
  ${V3_DIR}/V3Response.hpp
  )

install (FILES ${V3_INCLUDES}
               DESTINATION include/v3/include)

