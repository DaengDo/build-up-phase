cmake_minimum_required(VERSION 3.10)

project(hello_triangle)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR})

add_executable(${PROJECT_NAME} main.cpp)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_definitions(${PROJECT_NAME} PRIVATE NDEBUG)
    set(GLSLANG_LIB_NAMES 
        glslang 
        glslang-default-resource-limits 
        SPIRV-Tools 
        SPIRV-Tools-opt)
else()
    set(GLSLANG_LIB_NAMES 
        glslangd 
        glslang-default-resource-limitsd 
        SPIRV-Toolsd 
        SPIRV-Tools-optd)
endif()

# GLFW를 FetchContent로 가져오기
include(FetchContent)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
FetchContent_Declare(
  glfw
  GIT_REPOSITORY https://github.com/glfw/glfw.git
  GIT_TAG 3.4
)
FetchContent_MakeAvailable(glfw)

include_directories(
    $ENV{VULKAN_SDK}/Include
)

find_library(VULKAN_LIB vulkan-1 PATHS $ENV{VULKAN_SDK}/Lib)

set(GLSLANG_LIBS "")
foreach(LIB_NAME ${GLSLANG_LIB_NAMES})
    find_library(LIB_${LIB_NAME} ${LIB_NAME} PATHS $ENV{VULKAN_SDK}/Lib)
    if(LIB_${LIB_NAME})
        list(APPEND GLSLANG_LIBS ${LIB_${LIB_NAME}})
    else()
        message(FATAL_ERROR "Failed to find library: ${LIB_NAME}")
    endif()
endforeach()

target_link_libraries(${PROJECT_NAME} PRIVATE
    glfw
    ${VULKAN_LIB}
    ${GLSLANG_LIBS}
)
