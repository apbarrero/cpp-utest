cmake_minimum_required(VERSION 2.8)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

add_library(fizzbuzzer FizzBuzzer.hh FizzBuzzer.cc)
add_executable(fizzbuzz main.cc)
target_link_libraries(fizzbuzz fizzbuzzer)
