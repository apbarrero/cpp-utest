cmake_minimum_required(VERSION 2.8)

include(ExternalProject)

set(GMOCK_VERSION "1.7.0")
set(GMOCK_DIR "${CMAKE_CURRENT_BINARY_DIR}/gmock-${GMOCK_VERSION}")
ExternalProject_Add(project_gmock
    SVN_REPOSITORY http://googlemock.googlecode.com/svn/tags/release-${GMOCK_VERSION}
    PREFIX ${GMOCK_DIR}
    CMAKE_ARGS -DCMAKE_C_COMPILER:PATH=${CMAKE_C_COMPILER} -DCMAKE_CXX_COMPILER:PATH=${CMAKE_CXX_COMPILER}
    # Disable update step
    UPDATE_COMMAND ""
    # Disable install step
    INSTALL_COMMAND ""
)
ExternalProject_Get_Property(project_gmock source_dir)
ExternalProject_Get_Property(project_gmock binary_dir)

include_directories(${source_dir}/gtest/include)
add_library(gtest STATIC IMPORTED)
set_property(TARGET gtest PROPERTY IMPORTED_LOCATION ${binary_dir}/gtest/libgtest.a)
add_dependencies(gtest project_gmock)
add_library(gtest_main STATIC IMPORTED)
set_property(TARGET gtest_main PROPERTY IMPORTED_LOCATION ${binary_dir}/gtest/libgtest_main.a)
add_dependencies(gtest_main project_gmock)
include_directories(${source_dir}/include)
add_library(gmock STATIC IMPORTED)
set_property(TARGET gmock PROPERTY IMPORTED_LOCATION ${binary_dir}/libgmock.a)
add_dependencies(gmock project_gmock)
add_library(gmock_main STATIC IMPORTED)
set_property(TARGET gmock_main PROPERTY IMPORTED_LOCATION ${binary_dir}/libgmock_main.a)
add_dependencies(gmock_main project_gmock)

include_directories(${cpp_utest_SOURCE_DIR}/src)
set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++")
add_executable(utests StringCalculatorTest.cc)
target_link_libraries(utests sc gmock_main gmock gtest pthread)

set(utests_stamp
    "${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/utests.stamp")
add_custom_command(OUTPUT "${utests_stamp}"
    COMMAND ./utests
    COMMAND ${CMAKE_COMMAND} -E touch "${utests_stamp}"
    COMMENT "Running unit test"
    DEPENDS utests)
add_custom_target(check ALL DEPENDS "${utests_stamp}")

