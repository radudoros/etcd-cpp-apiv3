find_package(Boost REQUIRED COMPONENTS system thread locale random)
find_package(cpprestsdk REQUIRED)
find_package(gRPC CONFIG REQUIRED)
find_package(OpenSSL 1.0.0 REQUIRED)
find_package(Protobuf REQUIRED)

find_program(PROTOBUF_PROTOC protoc)
find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin)

include_directories(${Protobuf_INCLUDE_DIRS})

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
set_property(TARGET etcd-cpp-api PROPERTY CXX_STANDARD 17)

target_link_libraries(etcd-cpp-api ${OPENSSL_LIBRARIES} cpprestsdk::cpprest ${Boost_SYSTEM_LIBRARY} ${Protobuf_LIBRARIES} gRPC::grpc++)

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

