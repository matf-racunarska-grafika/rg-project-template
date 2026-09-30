cmake_minimum_required(VERSION 3.16)
set(PROJECT_NAME project)
project(${PROJECT_NAME} LANGUAGES C CXX)

# ---------------------------------------------------------------------------
# Language / build type
# ---------------------------------------------------------------------------
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Single-config generators (Make, Ninja): default to Release when unspecified.
# Multi-config generators (Visual Studio, Xcode): choose with --config.
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release RelWithDebInfo MinSizeRel)
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")

# ---------------------------------------------------------------------------
# System libraries: OpenGL and Assimp
# ---------------------------------------------------------------------------
# Prefer GLVND on Linux (modern Mesa / NVIDIA); ignored on Windows and macOS.
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)
find_package(Threads REQUIRED)

# Assimp: prefer the official CMake config (vcpkg, Homebrew, recent distros),
# fall back to the bundled cmake/modules/FindASSIMP.cmake.
find_package(assimp CONFIG QUIET)
if(TARGET assimp::assimp)
    message(STATUS "Assimp: using CMake config package (assimp::assimp)")
    set(ASSIMP_LINK_TARGET assimp::assimp)
else()
    message(STATUS "Assimp: config package not found, trying FindASSIMP.cmake")
    find_package(ASSIMP REQUIRED)
    set(ASSIMP_LINK_TARGET ${ASSIMP_LIBRARIES})
endif()

# Optional: FreeType (only if installed; used by some ImGui/text setups)
find_package(Freetype QUIET)

# ---------------------------------------------------------------------------
# Bundled libraries (libs/)
# ---------------------------------------------------------------------------
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)

add_subdirectory(libs/spdlog)
add_subdirectory(libs/glfw EXCLUDE_FROM_ALL)
add_subdirectory(libs/glad EXCLUDE_FROM_ALL)
add_subdirectory(libs/stb EXCLUDE_FROM_ALL)
add_subdirectory(libs/imgui EXCLUDE_FROM_ALL)
add_subdirectory(libs/glm EXCLUDE_FROM_ALL)

# ---------------------------------------------------------------------------
# Executable
# ---------------------------------------------------------------------------
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS
        "${CMAKE_SOURCE_DIR}/app/*.cpp"
        "${CMAKE_SOURCE_DIR}/app/*.c")
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS
        "${CMAKE_SOURCE_DIR}/app/*.h"
        "${CMAKE_SOURCE_DIR}/app/*.hpp")

add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_SOURCE_DIR}/app")
if(EXISTS "${CMAKE_SOURCE_DIR}/include")
    target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_SOURCE_DIR}/include")
endif()
if(DEFINED ASSIMP_INCLUDE_DIRS)
    target_include_directories(${PROJECT_NAME} PRIVATE ${ASSIMP_INCLUDE_DIRS})
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE
        glfw
        glad
        spdlog
        glm
        stb
        imgui
        OpenGL::GL
        Threads::Threads
        ${CMAKE_DL_LIBS}
        ${ASSIMP_LINK_TARGET})

if(Freetype_FOUND)
    target_link_libraries(${PROJECT_NAME} PRIVATE Freetype::Freetype)
endif()

# Absolute path to resources (shaders, textures, models) for use in code
target_compile_definitions(${PROJECT_NAME} PRIVATE
        RESOURCES_DIR="${CMAKE_SOURCE_DIR}/resources"
        SPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_TRACE
        SPDLOG_DEBUG_ON
        SPDLOG_TRACE_ON)

# ---------------------------------------------------------------------------
# Per-compiler / per-platform settings
# ---------------------------------------------------------------------------
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /permissive- /utf-8 /wd4100 /wd4189)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
            NOMINMAX
            _USE_MATH_DEFINES
            _CRT_SECURE_NO_WARNINGS)
    # Open in Visual Studio with this target as the startup project,
    # and run from the project root so relative resource paths work.
    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
    set_target_properties(${PROJECT_NAME} PROPERTIES
            VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}")
else()
    # GCC / Clang / AppleClang / MinGW
    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -Wno-unused-variable -Wno-unused-parameter)
endif()

if(APPLE)
    # Apple deprecated OpenGL (max 4.1 core); silence the warnings.
    target_compile_definitions(${PROJECT_NAME} PRIVATE GL_SILENCE_DEPRECATION)
endif()

# Put the executable in the project root on every platform.
# The $<1:...> generator expression stops Visual Studio / Xcode from appending
# a per-configuration subfolder (Debug/, Release/).
set_target_properties(${PROJECT_NAME} PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY "$<1:${CMAKE_SOURCE_DIR}>")

