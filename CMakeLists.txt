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

INCLUDE_DIRECTORIES(include ck/include)

SET(CMAKE_C_FLAGS "-Wpadded -Werror -Wall -O3 --std=c99")
SET(CMAKE_C_FLAGS_DEBUG "-g")

ADD_LIBRARY(softheap
    SHARED
    src/fifo.c
    #src/softheap.c
    ck/lib/libck.a
)

ADD_DEPENDENCIES(softheap concurrency-kit)
TARGET_LINK_LIBRARIES(softheap)