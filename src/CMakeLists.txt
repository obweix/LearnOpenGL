cmake_minimum_required(VERSION 3.0.0)

message(STATUS "src This is PROJECT_BINARY_DIR dir " ${PROJECT_BINARY_DIR})
message(STATUS "src This is PROJECT_SOURCE_DIR dir " ${PROJECT_SOURCE_DIR})
message(STATUS "src This is CMAKE_SOURCE_DIR dir " ${CMAKE_SOURCE_DIR})
message(STATUS "The PROJECT_BINARY_DIR is : " ${PROJECT_BINARY_DIR})
message(STATUS "CMAKE_INSTALL_PREFIX is : " ${CMAKE_INSTALL_PREFIX})

# 定义工程根目录; CMAKE_SOURCE_DIR为内建变量，表示工程根目录的CMakeLists.txt文件路径
set(ROOT_DIR ${CMAKE_SOURCE_DIR})

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

set(CMAKE_CXX_STANDARD 17)

# todo:将着色器程序安装到程序的执行目录
# aux_source_directory(${ROOT_DIR}/assets assets)
# install(DIRECTORY ${ROOT_DIR}/assets/ DESTINATION ${PROJECT_BINARY_DIR}/bin)

# 将着色器程序复制到程序的执行目录

#[[
Function declaring a target named ${targetName_} that carries dependency to files listed in
argument FILES, copied into directory OUTPUT_DIRECTORY. By default changes to the FILES are
tracked - changing any of the FILES will prompt an update. NO_DEPENDS flag can be given to
disable the tracking - this will only cause the files to be copied if they don't already exist at
the OUTPUT_DIRECTORY
Usage:

-- For scripts, etc. that need to be monitored for changes use the following form:
CopyIfDifferent(CopyShaders
    OUTPUT_DIRECTORY some/output/directory/assets/shaders
    FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders/shader.vs
        ${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders/shader.fs
)

-- For files that won't be updated and only need to be copied once add NO_DEPENDS flag
CopyIfDifferent(CopyTextures
    OUTPUT_DIRECTORY some/output/directory/assets/textures
    FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/assets/textures/awesomeface.png
    NO_DEPENDS
)

These will create `CopyShaders` and `CopyTextures` targets accordingly. These targets can then
be added as a dependency to the library/executable to make sure they're copied where they belong

add_dependencies(MyExecutable CopyShaders CopyTextures)
]]

function(CopyIfDifferent targetName_)
    set(options_ NO_DEPENDS)
    set(kwargs_ OUTPUT_DIRECTORY)
    set(list_args_ FILES)
    cmake_parse_arguments(arg "${options_}" "${kwargs_}" "${list_args_}" ${ARGN})
    foreach(unparsed_arg IN LISTS arg_UNPARSED_ARGUMENTS)
        message(WARNING "${ColorYellow}Unparsed argument: ${unparsed_arg}${ColorReset}")
    endforeach()
    if (NOT arg_FILES)
        message(FATAL_ERROR "${ColorRed}Error: Specify at least one FILES argument${ColorReset}")
    endif()

    get_filename_component(output_directory_ ${arg_OUTPUT_DIRECTORY}
        ABSOLUTE ${CMAKE_CURRENT_BINARY_DIR}
    )
    # message(STATUS "output_directory_ = ${output_directory_}")
    set(outputs_)
    foreach(file_ IN LISTS arg_FILES)
        get_filename_component(file_name_ "${file_}" NAME)
        list(APPEND outputs_ "${output_directory_}/${file_name_}")
    endforeach()

    add_custom_command(OUTPUT ${outputs_} ${CMAKE_COMMAND} -E copy_if_different)
    foreach(input_ output_ IN ZIP_LISTS arg_FILES outputs_)
        if(NOT arg_NO_DEPENDS)
            add_custom_command(OUTPUT ${outputs_}
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                    ${input_} ${output_}
                DEPENDS ${input_}
                APPEND
            )
        else()
            add_custom_command(OUTPUT ${outputs_}
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                    ${input_} ${output_}
                APPEND
            )
        endif()
    endforeach()

    add_custom_target(${targetName_} DEPENDS ${outputs_})

    # message(STATUS "FILES = ${arg_FILES}")
    # message(STATUS "OUTPUTS = ${outputs_}")
endfunction()


# 指定头文件搜索路径
include_directories(${ROOT_DIR}/lib)
include_directories(${ROOT_DIR}/include)


# 指定引用的外部库的搜索路径
link_directories(${ROOT_DIR}/lib/lib-arm64)

# add_executable(LearnOpenGL main.cpp glad.c)
aux_source_directory(./ src)
add_executable(LearnOpenGL ${src})
target_link_libraries(LearnOpenGL libglfw.3.dylib) 

CopyIfDifferent(CopyShaders
    OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin
    FILES
        ${ROOT_DIR}/assets/3.3.shader.vs
        ${ROOT_DIR}/assets/3.3.shader.fs
        ${ROOT_DIR}/assets/4.1.texture.fs
        ${ROOT_DIR}/assets/4.1.texture.vs
        ${ROOT_DIR}/assets/6.1.coordinate_systems.fs
        ${ROOT_DIR}/assets/6.1.coordinate_systems.vs
        ${ROOT_DIR}/assets/2.1.basic_lighting.vs
        ${ROOT_DIR}/assets/2.1.basic_lighting.fs
        ${ROOT_DIR}/assets/2.1.light_cube.vs
        ${ROOT_DIR}/assets/2.1.light_cube.fs
        ${ROOT_DIR}/assets/4.2.lighting_maps.vs
        ${ROOT_DIR}/assets/4.2.lighting_maps.fs
        ${ROOT_DIR}/assets/4.2.light_cube.vs
        ${ROOT_DIR}/assets/4.2.light_cube.fs
)
add_dependencies(LearnOpenGL CopyShaders)
