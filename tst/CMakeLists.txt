
add_executable(etcd_test EtcdTest.cpp EtcdSyncTest.cpp WatcherTest.cpp)
set_property(TARGET etcd_test PROPERTY CXX_STANDARD 11)

target_link_libraries(etcd_test etcd-cpp-api)

# Skip Building tests
# add_test(etcd_test etcd_test)
