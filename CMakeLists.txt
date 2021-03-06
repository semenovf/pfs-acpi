cmake_minimum_required (VERSION 3.1)
project(pfs-acpi C CXX)

option(pfs-acpi_BUILD_DEMO "Build Demo" OFF)

set(_acpi_interface_str)
set(SOURCES)
list(APPEND INCLUDE_DIRS
    "${CMAKE_CURRENT_LIST_DIR}/include"
    "${CMAKE_CURRENT_LIST_DIR}/3rdparty")

if (UNIX AND NOT APPLE) # AND NOT CYGWIN
    set(LINUX TRUE)
endif()

if (LINUX)
    if (EXISTS /sys/class)
        set(PFS_ACPI_SYS_INTERFACE TRUE)
        list(APPEND SOURCES "${CMAKE_CURRENT_LIST_DIR}/src/acpi_linux.cpp")
        set(_acpi_interface_str "Linux/sys")
    else ()
        set(PFS_ACPI_PROC_INTERFACE TRUE)
#         set(LIBACPI_ROOT "${CMAKE_CURRENT_LIST_DIR}/3rdparty/libacpi")
#         list(APPEND SOURCES "${LIBACPI_ROOT}/libacpi.c" "${LIBACPI_ROOT}/list.c")
        message(FATAL_ERROR "Unsupported yet: use libacpi instead")
        set(_acpi_interface_str "Linux/proc")
    endif()
elseif(WIN32)
    list(APPEND SOURCES "${CMAKE_CURRENT_LIST_DIR}/src/acpi_win32.cpp")
    set(_acpi_interface_str "Win32")
else()
    list(APPEND SOURCES "${CMAKE_CURRENT_LIST_DIR}/src/acpi_none.cpp")
    set(_acpi_interface_str "none")
endif()

message(STATUS "ACPI interface: " ${_acpi_interface_str})

list(REMOVE_DUPLICATES INCLUDE_DIRS)

add_library(${PROJECT_NAME} ${SOURCES})
add_library(pfs::acpi ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME} PUBLIC ${INCLUDE_DIRS})

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if (PFS_ACPI_SYS_INTERFACE)
        target_compile_definitions(${PROJECT_NAME} PRIVATE "-DPFS_ACPI_SYS_INTERFACE=1")
    elseif(PFS_ACPI_PROC_INTERFACE)
        target_compile_options(${PROJECT_NAME} PRIVATE -Wno-format-truncation)
        target_compile_definitions(${PROJECT_NAME} PRIVATE "-DPFS_ACPI_PROC_INTERFACE=1")
    endif()
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)

if (pfs-acpi_BUILD_DEMO)
    add_subdirectory(demo)
endif()
