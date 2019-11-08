project(cppmath_utests CXX)

include(../cmake/utils.cmake)

find_package(PkgConfig)
if(PkgConfig_FOUND)
  pkg_check_modules(systemd libsystemd)
endif()

set(CPPMATH_UTESTS_SOURCES
  main.cpp)

if(NOT CPPMATH_NO_EXCEPTIONS)
  # list(APPEND CPPMATH_UTESTS_SOURCES test_errors.cpp)
endif()

enable_testing()

function(cppmath_prepare_test test_target cppmath_lib)
  add_executable(${test_target} ${CPPMATH_UTESTS_SOURCES})
  cppmath_enable_warnings(${test_target})
  target_link_libraries(${test_target} PRIVATE ${cppmath_lib})
  if(systemd_FOUND)
    target_link_libraries(${test_target} PRIVATE ${systemd_LIBRARIES})
  endif()
  if(CPPMATH_SANITIZE_ADDRESS)
    cppmath_enable_sanitizer(${test_target})
  endif()
  add_test(NAME ${test_target} COMMAND ${test_target})
endfunction()

# The compiled library tests
if(CPPMATH_BUILD_TESTS)
  cppmath_prepare_test(cppmath-utests cppmath::cppmath)
endif()

# The header-only library version tests
if(CPPMATH_BUILD_TESTS_HO)
  cppmath_prepare_test(cppmath-utests-ho cppmath::cppmath_header_only)
endif()