Cmake_minimum_required(VERSION 3.12)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" ON)
option(BUILD_DOCS "Build documentation" OFF)
option(BUILD_COVERAGE "Build code coverage" OFF)

set(
        HUNTER_CACHE_SERVERS
        "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
        CACHE STRING "Default cache server"
)

include("cmake/HunterGate.cmake")

HunterGate(
        URL "https://github.com/cpp-pm/hunter/archive/v0.23.278.tar.gz"
        SHA1 "35fa55bc1dfcf9f4d7264f8bb26cd1759ce1ae07"
)



project(lab-06-multithreads)
string(APPEND CMAKE_CXX_FLAGS " -pedantic -Werror -Wall -Wextra -pthread -std=c++11")
string(APPEND CMAKE_CXX_FLAGS " -Wno-unused-command-line-argument")
string(APPEND CMAKE_CXX_FLAGS " -Wshadow -Wnon-virtual-dtor")

hunter_add_package(nlohmann_json)
find_package(nlohmann_json CONFIG REQUIRED)

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

hunter_add_package(Boost COMPONENTS filesystem log system thread)
find_package(Boost CONFIG REQUIRED filesystem log log_setup system thread)



include_directories(${Boost_INCLUDE_DIRS})
add_library(template STATIC
        # enum your files and delete this comment
        ${CMAKE_CURRENT_SOURCE_DIR}/sources/Hash.cpp
        include/Hash.hpp
        sources/main.cpp)
add_executable(tests

        ${CMAKE_CURRENT_SOURCE_DIR}/tests/test.cpp
        include/Hash.hpp
        sources/Hash.cpp
        )

add_executable(main
        ${CMAKE_CURRENT_SOURCE_DIR}/sources/main.cpp
        include/Hash.hpp
        sources/Hash.cpp)


target_include_directories(template
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/third-party/PicoSHA2
        )
target_include_directories(tests
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )
target_include_directories(main
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )


target_link_libraries(
        template
        Boost::log_setup
        Boost::log
        Boost::thread
        Boost::filesystem
        Boost::system
        nlohmann_json::nlohmann_json
)
target_link_libraries(tests GTest::gmock_main  Boost::log_setup
        Boost::log
        Boost::thread
        Boost::filesystem
        Boost::system
        nlohmann_json::nlohmann_json template)
target_link_libraries(main Boost::log_setup
        Boost::log
        Boost::thread
        Boost::filesystem
        Boost::system
        nlohmann_json::nlohmann_json Boost::system)

enable_testing()
add_test(NAME unit_tests COMMAND tests)

if(BUILD_COVERAGE)
  set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
  find_package(codecov)

  list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/tests/*'")
  coverage_evaluate()
endif()