cmake_minimum_required(VERSION 3.16)
project(libslock)

set(LIBRARY_TARGET_NAME ${PROJECT_NAME})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_VERBOSE_MAKEFILE ON)

file(GLOB LIBSLOCK_SRC_GLOB_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")
set_source_files_properties(${LIBSLOCK_SRC_GLOB_FILES} PROPERTIES LANGUAGE CXX)

if (CMAKE_BUILD_TYPE EQUAL "DEBUG")
    set(DEBUG_FLAGS "-ggdb -DDEBUG")
    set(COMPILE_FLAGS "-fno-inline")
    set(CMAKE_CXX_FLAGS_DEBUG "-O0")
else()
    set(DEBUG_FLAGS "-Wall")
    set(CMAKE_CXX_FLAGS_RELEASE "-O2")
endif()

include(ProcessorCount)
ProcessorCount(N)
if(NOT N EQUAL 0)
    add_compile_definitions(CORE_NUM=${N})
else()
    add_compile_definitions(CORE_NUM=${8})
endif()

add_compile_definitions(ADD_PADDING LINUXOS)
set(CMAKE_CXX_FLAGS "-Wall ${DEBUG_FLAGS} ${COMPILE_FLAGS}")

set(${LIBRARY_TARGET_NAME}_HDR
        include/alock.h
        include/atomic_ops.h
        include/clh.h
        include/gl_lock.h
        include/hclh.h
        include/htlock.h
        include/lock_if.h
        include/mcs.h
        include/platform_defs.h
        include/rw_ttas.h
        include/spinlock.h
        include/ticket.h
        include/ttas.h
        include/utils.h
)
set(${LIBRARY_TARGET_NAME}_SRC
        src/alock.c
        src/clh.c
        src/gl_lock.c
        src/hclh.c
        src/htlock.c
        src/mcs.c
        src/rw_ttas.c
        src/spinlock.c
        src/ticket.c
        src/ttas.c
)

add_library(${LIBRARY_TARGET_NAME} ${${LIBRARY_TARGET_NAME}_SRC} ${${LIBRARY_TARGET_NAME}_HDR})
add_library(${PROJECT_NAME}::${LIBRARY_TARGET_NAME} ALIAS ${LIBRARY_TARGET_NAME})

set_target_properties(${LIBRARY_TARGET_NAME} PROPERTIES PUBLIC_HEADER "${${LIBRARY_TARGET_NAME}_HDR}")

target_include_directories(${LIBRARY_TARGET_NAME} PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")

target_link_libraries(${LIBRARY_TARGET_NAME}
    pthread
    rt
    numa
)
message(STATUS "Created target ${LIBRARY_TARGET_NAME} for export ${PROJECT_NAME}.")