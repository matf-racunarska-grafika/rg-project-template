cmake_minimum_required(VERSION 3.21)

add_library(stb stb.cpp)

set_source_files_properties(stb_image.cpp include/stb_image.h
        PROPERTIES
        COMPILE_FLAGS
        "-Wno-shift-negative-value -Wno-implicit-fallthrough")

target_include_directories(stb PUBLIC include/)
