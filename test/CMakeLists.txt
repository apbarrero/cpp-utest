cmake_minimum_required(VERSION 2.8)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

find_package(Threads REQUIRED)

include(ExternalProject)
ExternalProject_Add(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.8.0
  CMAKE_ARGS -DCMAKE_C_COMPILER:PATH=${CMAKE_C_COMPILER} -DCMAKE_CXX_COMPILER:PATH=${CMAKE_CXX_COMPILER}
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
  LOG_DOWNLOAD ON
  LOG_CONFIGURE ON
  LOG_BUILD ON)

ExternalProject_Get_Property(googletest source_dir)
set(GTEST_INCLUDE_DIRS ${source_dir}/googletest/include)
set(GMOCK_INCLUDE_DIRS ${source_dir}/googlemock/include)
include_directories(${GTEST_INCLUDE_DIRS} ${GMOCK_INCLUDE_DIRS})

# The cloning of the above repo doesn't happen until make, however if the dir doesn't
# exist, INTERFACE_INCLUDE_DIRECTORIES will throw an error.
# To make it work, we just create the directory now during config.
file(MAKE_DIRECTORY ${GTEST_INCLUDE_DIRS})
file(MAKE_DIRECTORY ${GMOCK_INCLUDE_DIRS})

ExternalProject_Get_Property(googletest binary_dir)
set(GTEST_LIBRARY_PATH ${binary_dir}/googlemock/gtest/${CMAKE_FIND_LIBRARY_PREFIXES}gtest.a)
set(GTEST_LIBRARY gtest)
add_library(${GTEST_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GTEST_LIBRARY} PROPERTIES
    "IMPORTED_LOCATION" "${GTEST_LIBRARY_PATH}"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
    "INTERFACE_INCLUDE_DIRECTORIES" "${GTEST_INCLUDE_DIRS}")
add_dependencies(${GTEST_LIBRARY} googletest)

set(GTEST_MAIN_LIBRARY_PATH ${binary_dir}/googlemock/gtest/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main.a)
set(GTEST_MAIN_LIBRARY gtest_main)
add_library(${GTEST_MAIN_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GTEST_MAIN_LIBRARY} PROPERTIES
    "IMPORTED_LOCATION" "${GTEST_MAIN_LIBRARY_PATH}"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
    "INTERFACE_INCLUDE_DIRECTORIES" "${GTEST_INCLUDE_DIRS}")
add_dependencies(${GTEST_MAIN_LIBRARY} googletest)

set(GMOCK_LIBRARY_PATH ${binary_dir}/googlemock/${CMAKE_FIND_LIBRARY_PREFIXES}gmock.a)
set(GMOCK_LIBRARY gmock)
add_library(${GMOCK_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GMOCK_LIBRARY} PROPERTIES
    "IMPORTED_LOCATION" "${GMOCK_LIBRARY_PATH}"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
    "INTERFACE_INCLUDE_DIRECTORIES" "${GMOCK_INCLUDE_DIRS}")
add_dependencies(${GMOCK_LIBRARY} googletest)

set(GMOCK_MAIN_LIBRARY_PATH ${binary_dir}/googlemock/${CMAKE_FIND_LIBRARY_PREFIXES}gmock_main.a)
set(GMOCK_MAIN_LIBRARY gmock_main)
add_library(${GMOCK_MAIN_LIBRARY} UNKNOWN IMPORTED)
set_target_properties(${GMOCK_MAIN_LIBRARY} PROPERTIES
    "IMPORTED_LOCATION" "${GMOCK_MAIN_LIBRARY_PATH}"
    "IMPORTED_LINK_INTERFACE_LIBRARIES" "${CMAKE_THREAD_LIBS_INIT}"
    "INTERFACE_INCLUDE_DIRECTORIES" "${GMOCK_INCLUDE_DIRS}")
add_dependencies(${GMOCK_MAIN_LIBRARY} ${GTEST_LIBRARY})

include_directories(${cpp_utest_SOURCE_DIR}/src)
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
   set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++")
endif()
#add_executable(utests )
#target_link_libraries(utests gmock_main gmock gtest pthread)
