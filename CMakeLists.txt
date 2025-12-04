CMAKE_MINIMUM_REQUIRED(VERSION 3.23)

project(ryzenadj VERSION 0.18 DESCRIPTION "RyzenAdj library - PowerTuner fork" HOMEPAGE_URL "https://github.com/PowerTuner/RyzenAdj" LANGUAGES C CXX)

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

string(TOLOWER ${CMAKE_BUILD_TYPE} build_type)
set(PROJECT_AUTHOR "PowerTuner fork: kylon, Original dev: FlyGoat")
set(LINK_LIBS "")

set(PROJECT_SOURCES
    libryzenadj/osdep.h
    libryzenadj/smu_access.h
    libryzenadj/smu_access.c
    libryzenadj/ryzen_opts.h
    libryzenadj/ryzen_opts.c
    libryzenadj/ryzenadj.c
)

set(PUB_HEADERS
    libryzenadj/ryzenadj.h
)

if (WIN32)
    configure_file(win32/win.rc.in ${CMAKE_CURRENT_SOURCE_DIR}/win.rc)

    list(APPEND LINK_LIBS
        WinRing0x64
    )

    list(APPEND PROJECT_SOURCES
        libryzenadj/win32/osdep_win32.cpp
        win.rc
    )
elseif (LINUX)
    list(APPEND LINK_LIBS
        pci
    )

    list(APPEND PROJECT_SOURCES
        libryzenadj/linux/osdep_linux_mem.h
        libryzenadj/linux/osdep_linux_mem.c
        libryzenadj/linux/osdep_linux_smu_kernel_module.h
        libryzenadj/linux/osdep_linux_smu_kernel_module.c
        libryzenadj/linux/osdep_linux.c
    )
else ()
    message(FATAL_ERROR "!Unsupported OS!")
endif ()

add_library(${PROJECT_NAME} ${PUB_HEADERS} ${PROJECT_SOURCES})
add_library("RYADJ::RyzenAdjLib" ALIAS ${PROJECT_NAME})

if (build_type MATCHES debug)
    message(STATUS "${PROJECT_NAME}: debug prints are enabled")
    target_compile_definitions(${PROJECT_NAME} PRIVATE RYZENADJ_DEBUG)
endif ()

include(CheckIPOSupported)
check_ipo_supported(RESULT has_ipo OUTPUT error)
if (has_ipo)
    message(STATUS "${PROJECT_NAME}: IPO / LTO enabled")
    set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

if (WIN32)
    set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME "libryzenadj")
    target_link_directories(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/win32")
endif ()

target_compile_definitions(${PROJECT_NAME} PUBLIC LIBRYZENADJ_EXPORT)
target_link_libraries(${PROJECT_NAME} ${LINK_LIBS})
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

target_sources(${PROJECT_NAME} PUBLIC
    FILE_SET rylib_headers
    TYPE HEADERS
    FILES ${PUB_HEADERS}
)

include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
    FILE_SET rylib_headers DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
