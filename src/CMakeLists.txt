cmake_minimum_required(VERSION 3.0.0)

message(STATUS "src This is PROJECT_BINARY_DIR dir " ${PROJECT_BINARY_DIR})
message(STATUS "src This is PROJECT_SOURCE_DIR dir " ${PROJECT_SOURCE_DIR})
message(STATUS "src This is CMAKE_SOURCE_DIR dir " ${CMAKE_SOURCE_DIR})

# 定义工程根目录; CMAKE_SOURCE_DIR为内建变量，表示工程根目录的CMakeLists.txt文件路径
set(ROOT_DIR ${CMAKE_SOURCE_DIR})

# 指定头文件搜索路径
include_directories(${ROOT_DIR}/lib)

# 指定引用的外部库的搜索路径
link_directories(${ROOT_DIR}/lib/lib-arm64)

add_executable(LearnOpenGL main.cpp glad.c)

target_link_libraries(LearnOpenGL libglfw.3.dylib) 
 