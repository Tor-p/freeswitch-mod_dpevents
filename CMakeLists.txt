cmake_minimum_required(VERSION 3.18)
project(mod_dpevents
        VERSION 1.0.3
        DESCRIPTION "FreeSWITCH module for events bindings from dialplan."
        HOMEPAGE_URL "https://github.com/kvishnivetsky/freeswitch-mod_dpevents")

include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")

option(ENABLE_LOCAL "Enable local debug-specific features and setting channel variables" OFF)

if(ENABLE_LOCAL)
    set(ENV{PKG_CONFIG_PATH} "/usr/local/freeswitch/lib/pkgconfig:$ENV{PKG_CONFIG_PATH}")
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

find_package(PkgConfig REQUIRED)

pkg_check_modules(FreeSWITCH REQUIRED IMPORTED_TARGET freeswitch)
pkg_get_variable(FS_MOD_DIR freeswitch modulesdir)
message(STATUS "FreeSWITCH modules dir: ${FS_MOD_DIR}")

add_library(mod_dpevents SHARED mod_dpevents.c)

set_property(TARGET mod_dpevents PROPERTY POSITION_INDEPENDENT_CODE ON)

target_link_libraries(mod_dpevents PRIVATE PkgConfig::FreeSWITCH pthread)

if (ENABLE_LOCAL)
    target_compile_definitions(mod_dpevents PRIVATE ENABLE_LOCAL)
endif()

set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6, libfreeswitch1")
set(CPACK_PACKAGE_NAME "mod-dpevents")

install(TARGETS ${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
        DESTINATION ${FS_MOD_DIR})

message(STATUS "Components to pack: ${CPACK_COMPONENTS_ALL}")

include(Packing)
