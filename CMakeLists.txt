cmake_minimum_required(VERSION 2.8.9)
project(pkcs11test)

set(VERSION "0.1.0")
set(VERSION_MAJOR 0)
set(VERSION_MINOR 1)
set(VERSION_PATCH 0)

# Download and unpack googletest at configure time
configure_file(CMakeLists-googletest.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

set(gtest_build_samples OFF CACHE INTERNAL "" FORCE)
set(gtest_build_tests OFF CACHE INTERNAL "" FORCE)
set(gtest_disable_pthreads OFF CACHE INTERNAL "" FORCE)
set(gtest_force_shared_crt ON CACHE INTERNAL "" FORCE)
set(INSTALL_GTEST OFF CACHE INTERNAL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE INTERNAL "" FORCE)
set(BUILD_GMOCK OFF CACHE INTERNAL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
                 ${CMAKE_CURRENT_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)


option(STRICT_P11 "Define STRICT_P11 somewhere to force 1-byte alignment on P11 structures")

file(GLOB PKCS11TEST_SRC RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "*.cc")

# Add the basic sources to the library
add_executable(pkcs11test ${PKCS11TEST_SRC} )

if(STRICT_P11)
target_compile_definitions(pkcs11test PRIVATE -DSTRICT_P11)
endif()

if(WIN32)
set(BUILD_TESTS OFF CACHE INTERNAL "" FORCE)
add_subdirectory(third_party/dlfcn-win32)
set(CMAKE_DLFCN_LIBS dl)
include_directories(pkcs11test third_party/dlfcn-win32 third_party/getopt)
endif()

include_directories(pkcs11test ${CMAKE_CURRENT_SOURCE_DIR} third_party/pkcs11)

target_link_libraries(pkcs11test gtest ${CMAKE_DLFCN_LIBS})

