cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_COVERAGE "Build coverage" OFF)

set(
  HUNTER_CACHE_SERVERS
  "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
  CACHE STRING "Default cache server"
)

include("tools/gate/cmake/HunterGate.cmake")

HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.314.tar.gz"
  SHA1 "95c47c92f68edb091b5d6d18924baabe02a6962a"
)

project(app-bugs-world)

if(WIN32)
else()
  set(BUILD_SSL NO)
  string(APPEND CMAKE_CXX_FLAGS " -pthread")
  #string(APPEND CMAKE_CXX_FLAGS " -pedantic -Werror -Wall -Wextra")
  #string(APPEND CMAKE_CXX_FLAGS " -Wno-unused-command-line-argument")
  #string(APPEND CMAKE_CXX_FLAGS " -Wshadow -Wnon-virtual-dtor")
endif()

project(app)

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

hunter_add_package(cxxopts)
find_package(cxxopts CONFIG REQUIRED)

hunter_add_package(nlohmann_json)
find_package(nlohmann_json CONFIG REQUIRED)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/third-party/restbed restbed)

add_library(bw-core-lib STATIC
        sources/messages.cpp)
add_executable(tests
        tests/matrix_ut.cpp)
add_executable(app
        sources/main.cpp sources/details/http_service.cpp sources/details/app.cpp)

target_include_directories(bw-core-lib
  PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
)
target_include_directories(tests
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        )
target_include_directories(app
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/third-party/restbed/source
        )

target_link_libraries(app bw-core-lib restbed-static cxxopts::cxxopts nlohmann_json::nlohmann_json)
target_link_libraries(tests bw-core-lib GTest::gmock_main )

enable_testing()
add_test(NAME unit_tests COMMAND tests)

if(BUILD_COVERAGE)
  function(ENABLE_PROJECT_COVERAGE PROJECT)
    target_compile_options(${PROJECT} PRIVATE --coverage)
    target_link_libraries(${PROJECT} --coverage)
    add_coverage(${PROJECT})
  endfunction()

  set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
  find_package(codecov)
  # TODO: change template word to project name and delete this comment
  enable_project_coverage(template)
  enable_project_coverage(tests)
  list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/tests/*'")
  coverage_evaluate()
endif()
