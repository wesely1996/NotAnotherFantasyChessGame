cmake_minimum_required(VERSION 3.20)

project(ApawnstaleGame
    VERSION 0.1.0
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(FetchContent)

set(FETCHCONTENT_QUIET FALSE)

FetchContent_Declare(
    raylib
    GIT_REPOSITORY https://github.com/raysan5/raylib.git
    GIT_TAG 5.5
    GIT_SHALLOW TRUE
)

set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(BUILD_GAMES OFF CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(raylib)

FetchContent_Declare(
    json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.3
    GIT_SHALLOW TRUE
)

set(JSON_BuildTests OFF CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(json)

add_executable(apawnstale
    src/main.cpp
)

target_link_libraries(apawnstale PRIVATE raylib nlohmann_json::nlohmann_json)

if (MINGW)
    # This toolchain's ld (GNU ld 2.43.1, MSYS2 UCRT64) fails to link any
    # object file with C++ exception-handling unwind tables against the
    # dynamic libgcc_s_seh-1.dll/libstdc++-6.dll ("ld returned 116 exit
    # status", no diagnostic). Reproduces with a bare throw/catch, no
    # json involved. Statically linking the runtime avoids the bad path.
    target_link_options(apawnstale PRIVATE -static-libgcc -static-libstdc++)
endif()
