cmake_minimum_required(VERSION 3.5)

project(libudev-cmake LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(TEST "build unit test" OFF)
option(AS_CHILD_PROJECT "as child project" ON)

if (AS_CHILD_PROJECT)
    set(HAS_THIRDPARTY_LIBUDEV ON PARENT_SCOPE)
endif()

if (NOT UNIX)
    message(FATAL_ERROR "unsupporsted ${CMAKE_SYSTEM_NAME}")
endif()

function(libudevMain)
    if (NOT CMD_MAKE)
        message(STATUS "[libudev] set CMD_MAKE is make.")
        set(CMD_MAKE make)
    endif()

    add_library(thirdparty_libudev INTERFACE)
    set(root_dir src/libudev)
    set(SOURCES
        "${root_dir}/libudev-device-private.c"
        "${root_dir}/libudev.c"
        "${root_dir}/libudev-device.c"
        "${root_dir}/libudev-enumerate.c"
        "${root_dir}/libudev-list.c"
        "${root_dir}/libudev-monitor.c"
        "${root_dir}/libudev-queue.c"
        "${root_dir}/libudev-util.c"
        "${root_dir}/libudev-queue-private.c"
    )
    add_library(thirdparty_libudev_static ${SOURCES})
    target_compile_definitions(thirdparty_libudev_static PRIVATE
        "FIRMWARE_PATH=\"/lib/firmware/updates/\", \"/lib/firmware/\""
        HAVE_CONFIG_H
        LIBEXECDIR=\"/lib/udev\"
        SYSCONFDIR=\"/etc\"
    )
    target_include_directories(thirdparty_libudev_static PRIVATE ${root_dir} ${CMAKE_CURRENT_SOURCE_DIR}/config)
    target_compile_options(thirdparty_libudev_static PRIVATE -include config.h)

    target_link_libraries(thirdparty_libudev PUBLIC INTERFACE thirdparty_libudev_static)
    target_include_directories(thirdparty_libudev INTERFACE SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/config)
endfunction(libudevMain)

function(testMain)
    # test-libudev.c
endfunction(testMain)

function(testMultiThreadMain)
    # test-udev-device-thread.c
endfunction(testMultiThreadMain)

libudevMain()

if (TEST)
    testMain()
    testMultiThreadMain()
endif()
