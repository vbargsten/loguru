cmake_minimum_required(VERSION 3.0)
include(GNUInstallDirs)
set(CMAKE_CXX_STANDARD 11)

project(loguru VERSION 0.1 LANGUAGES CXX)

set(LOGURU_USE_FMTLIB ON CACHE STRING "Use fmtlib formatting.")
set_property(CACHE LOGURU_USE_FMTLIB PROPERTY TYPE BOOL)


set("${PROJECT_NAME}_HEADERS"
    loguru.hpp
)

add_library("${PROJECT_NAME}"
    loguru.cpp
)

set(LOGURU_USE_FMTLIB_DEF "0")
if (${LOGURU_USE_FMTLIB})
    set(LOGURU_USE_FMTLIB_DEF "1")
    find_package(fmt REQUIRED)
    target_link_libraries("${PROJECT_NAME}" fmt::fmt)
endif()

target_link_libraries("${PROJECT_NAME}" pthread dl)

target_compile_definitions("${PROJECT_NAME}" PUBLIC LOGURU_USE_FMTLIB=${LOGURU_USE_FMTLIB_DEF})

set_property(TARGET "${PROJECT_NAME}" PROPERTY POSITION_INDEPENDENT_CODE ON)

set_target_properties("${PROJECT_NAME}" 
    PROPERTIES PUBLIC_HEADER "${${PROJECT_NAME}_HEADERS}"
)

# Define headers for this library. PUBLIC headers are used for
# compiling the library, and will be added to consumers' build
# paths.
target_include_directories("${PROJECT_NAME}" PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
    PRIVATE .
)

install(TARGETS ${PROJECT_NAME} EXPORT "${PROJECT_NAME}Config"
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

# This makes the project importable from the install directory
# Put config file in per-project dir (name MUST match), can also
# just go into 'cmake'.
install(EXPORT "${PROJECT_NAME}Config" DESTINATION share/${PROJECT_NAME}/cmake)

# This makes the project importable from the build directory
export(TARGETS ${PROJECT_NAME} FILE ${PROJECT_NAME}.cmake)


