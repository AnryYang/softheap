CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
INCLUDE(ExternalProject)
PROJECT(softheap)

ExternalProject_Add(
    concurrency-kit
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/ck
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/ck
    CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR>
    BUILD_IN_SOURCE 1
    #INSTALL_COMMAND ""
    BUILD_COMMAND ${MAKE})

INCLUDE_DIRECTORIES(include ck/include lz4)

SET(CMAKE_C_FLAGS
    # Reenable when no more unused params
    #"-Werror -Wall -Wextra -pedantic -O3 --std=c99")
    "-Werror -Wall -fPIC -O3 --std=c99")
SET(CMAKE_C_FLAGS_DEBUG "-g")

ADD_LIBRARY(lz4
    STATIC
    lz4/lz4.c
    lz4/lz4hc.c
)

ADD_LIBRARY(softheap-static
    STATIC
    src/store/mmap.c
    src/store/lz4_store.c
    src/persistent_atomic_value.c
    src/segment_list/segment_list.c
    src/storage_manager/storage_manager.c
    #src/fifo.c
    #src/softheap.c
)

ADD_LIBRARY(softheap
    SHARED
    src/store/mmap.c
    src/store/lz4_store.c
    src/persistent_atomic_value.c
    src/segment_list/segment_list.c
    src/storage_manager/storage_manager.c
    #src/fifo.c
    #src/softheap.c
)

SET_TARGET_PROPERTIES(softheap
    PROPERTIES COMPILE_FLAGS "-Wpadded")
SET_TARGET_PROPERTIES(softheap-static
    PROPERTIES COMPILE_FLAGS "-Wpadded")

ADD_DEPENDENCIES(softheap lz4 concurrency-kit)
ADD_DEPENDENCIES(softheap-static lz4 concurrency-kit)
TARGET_LINK_LIBRARIES(softheap 
    lz4
    ${CMAKE_CURRENT_BINARY_DIR}/ck/lib/libck.so.0.4.3)
TARGET_LINK_LIBRARIES(softheap-static 
    lz4
    ${CMAKE_CURRENT_BINARY_DIR}/ck/lib/libck.a)

ADD_SUBDIRECTORY(test)
ENABLE_TESTING()
ADD_TEST(NAME test_lz4_store COMMAND test_lz4_store)
ADD_TEST(NAME test_mmap_store COMMAND test_mmap_store)
ADD_TEST(NAME test_store_threaded COMMAND test_store_threaded)
ADD_TEST(NAME test_persistent_atomic_value COMMAND test_persistent_atomic_value)
ADD_TEST(NAME test_segment_list_basic COMMAND test_segment_list_basic)
ADD_TEST(NAME test_segment_list_threaded COMMAND test_segment_list_threaded)
ADD_TEST(NAME test_storage_manager_basic COMMAND test_storage_manager_basic)
ADD_TEST(NAME test_storage_manager_threaded COMMAND test_storage_manager_threaded)
