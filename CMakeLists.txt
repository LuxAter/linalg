# ##############################################################################
# PROJECT
# ##############################################################################
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
project(
  linalg
  VERSION 0.0.6
  LANGUAGES CXX)

if(NOT DEFINED LINALG_MASTER_proj)
  if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(LINALG_MASTER_proj TRUE)
  else()
    set(LINALG_MASTER_proj FALSE)
  endif()
endif()

# ##############################################################################
# OPTIONS
# ##############################################################################
option(LINALG_BUILD_SHARED "Build shared library" OFF)
option(LINALG_BUILD_TESTS "Build tests" ${LINALG_MASTER_proj})
option(LINALG_BUILD_TESTS_HO "Build tests using the header only version"
       ${LINALG_MASTER_proj})
option(LINALG_TEST_COVERAGE "Enable test coverage reporting"
       ${LINALG_MASTER_proj})
option(LINALG_INSTALL "Generate the install target" ${LINALG_MASTER_proj})
option(STATIC_ANALYSIS "Enable static analysis" FALSE)

# ##############################################################################
# COMPILER
# ##############################################################################
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ##############################################################################
# SOURCE
# ##############################################################################
set(SOURCE src/linalg.cpp)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/include/${proj_NAME}/version.h.in"
               "${CMAKE_CURRENT_SOURCE_DIR}/include/${proj_NAME}/version.h")
set(LINALG_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(LINALG_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# ##############################################################################
# BUILD TYPE
# ##############################################################################
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE
      "Debug"
      CACHE STRING "Choose the type of build" FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif()
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)
  if(EXISTS "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json")
    execute_process(
      COMMAND ${CMAKE_COMMAND} -E copy_if_different
              ${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json
              ${CMAKE_CURRENT_SOURCE_DIR}/compile_commands.json)
  endif()
endif()

# ##############################################################################
# STATIC ANALYSIS
# ##############################################################################
if(STATIC_ANALYSIS OR CMAKE_BUILD_TYPE STREQUAL "Release")
  find_program(CLANG_TIDY "clang-tidy")
  if(CLANG_TIDY)
    set(CMAKE_CXX_CLANG_TIDY ${CLANG_TIDY})
  endif()
  find_program(CPPCHECK "cppcheck")
  if(CPPCHECK)
    set(CMAKE_CXX_CPPCHECK ${CPPCHECK})
    list(APPEND CMAKE_CXX_CPPCHECK "--force")
  endif()
  find_program(CPPLINT "cpplint")
  if(CPPLINT)
    set(CMAKE_CXX_CPPLINT ${CPPLINT})
  endif()
endif()

# ##############################################################################
# TARGET LIBRARY
# ##############################################################################
if(LINALG_BUILD_SHARED)
  add_library(linalg SHARED ${SOURCE})
else()
  add_library(linalg STATIC ${SOURCE})
endif()
add_library(${proj_NAME}::linalg ALIAS linalg)
target_include_directories(
  linalg PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
                "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")
target_compile_options(linalg PUBLIC -DLINALG_COMPILED_LIB)
target_compile_options(linalg
                       PRIVATE $<$<CONFIG:Debug>:-O0>$<$<CONFIG:Release>:-O3>)
target_compile_options(
  linalg
  PRIVATE
    $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>:
    -Wall
    -Wextra
    -Wconversion
    -pedantic
    -Wfatal-errors>
    $<$<CXX_COMPILER_ID:MSVC>:/W4
    /WX>)
target_compile_definitions(linalg PUBLIC
                           $<$<CONFIG:Debug>:DEBUG>$<$<CONFIG:Release>:RELEASE>)
set_target_properties(linalg PROPERTIES VERSION ${proj_VERSION} SOVERSION
                                        ${proj_VERSION_MAJOR})
install(TARGETS ${proj_NAME})

# ##############################################################################
# TARGET HEADER ONLY LIBRARY
# ##############################################################################
add_library(linalg_header_only INTERFACE)
add_library(${proj_NAME}::header_only ALIAS linalg_header_only)
target_include_directories(
  linalg_header_only
  INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
            "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

# ##############################################################################
# TESTING
# ##############################################################################
if(LINALG_BUILD_TESTS OR LINALG_BUILD_TESTS_HO)
  message(STATUS "Generating tests")
  enable_testing()
  add_subdirectory(tests)
endif()

# ##############################################################################
# PACKAGING
# ##############################################################################
if(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CPACK_GENERATOR "STGZ;TGZ;ZIP")
  set(CPACK_PACKAGE_VENDOR "LuxAter")
  set(CPACK_PACKAGE_VERSION_MAJOR ${proj_VERSION_MAJOR})
  set(CPACK_PACKAGE_VERSION_MINOR ${proj_VERSION_MINOR})
  set(CPACK_PACKAGE_VERSION_PATCH ${proj_VERSION_PATCH})
  set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
  set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.md")
  set(CPACK_MONOLITHIC_INSTALL TRUE)
  include(CPack)
endif()

# ------------------------------------------------------------------------------
# ---------
# Install
# ------------------------------------------------------------------------------
# ---------
# if(CPPMATH_INSTALL) message(STATUS "Generating install") set(proj_config_in
# "${CMAKE_CURRENT_LIST_DIR}/cmake/cppmathConfig.cmake.in") set(proj_config_out
# "${CMAKE_CURRENT_BINARY_DIR}/cppmathConfig.cmake") set(config_targets_file
# "cppmathConfigTargets.cmake") set(version_config_file
# "${CMAKE_CURRENT_BINARY_DIR}/cppmathConfigVersion.cmake") set(export_dest_dir
# "${CMAKE_INSTALL_LIBDIR}/cmake/cppmath") set(pkgconfig_install_dir
# "${CMAKE_INSTALL_LIBDIR}/pkgconfig") set(pkg_config
# "${CMAKE_BINARY_DIR}/${proj_NAME}.pc")
#
# # ----------------------------------------------------------------------------
# # ----------- # Include files #
# ---------------------------------------------------------------------------- #
# ----------- install(DIRECTORY include/ DESTINATION
# "${CMAKE_INSTALL_INCLUDEDIR}") install( TARGETS cppmath cppmath_header_only
# EXPORT cppmath DESTINATION "${CMAKE_INSTALL_LIBDIR}")
#
# # ----------------------------------------------------------------------------
# # ----------- # Install pkg-config file #
# ---------------------------------------------------------------------------- #
# ----------- get_target_property(PKG_CONFIG_DEFINES cppmath
# INTERFACE_COMPILE_DEFINITIONS) string(REPLACE ";" " -D" PKG_CONFIG_DEFINES
# "${PKG_CONFIG_DEFINES}") string(CONCAT PKG_CONFIG_DEFINES "-D"
# "${PKG_CONFIG_DEFINES}") configure_file("cmake/${proj_NAME}.pc.in"
# "${pkg_config}" @ONLY) install(FILES "${pkg_config}" DESTINATION
# "${pkgconfig_install_dir}")
#
# # ----------------------------------------------------------------------------
# # ----------- # Install CMake config files #
# ---------------------------------------------------------------------------- #
# ----------- install( EXPORT cppmath DESTINATION ${export_dest_dir} NAMESPACE
# cppmath:: FILE ${config_targets_file})
#
# include(CMakePackageConfigHelpers) configure_file("${proj_config_in}"
# "${proj_config_out}" @ONLY)
#
# write_basic_package_version_file("${version_config_file}" COMPATIBILITY
# SameMajorVersion) install(FILES "${proj_config_out}"
# "${version_config_file}" DESTINATION "${export_dest_dir}")
#
# # ----------------------------------------------------------------------------
# # ----------- # Support creation of installable packages #
# ---------------------------------------------------------------------------- #
# ----------- include(cmake/cppmathCPack.cmake)
#
# endif()
