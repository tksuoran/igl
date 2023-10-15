# Copyright (c) Meta Platforms, Inc. and affiliates.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

# CMake 3.16 can load Vulkan SDK
cmake_minimum_required(VERSION 3.16)

project(IGLVulkan CXX C)

file(GLOB SRC_FILES LIST_DIRECTORIES false RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp *.c util/*.cpp)
file(GLOB HEADER_FILES LIST_DIRECTORIES false RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.h util/*.h)

if(APPLE)
  file(GLOB APPLE_SRC_FILES LIST_DIRECTORIES false RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} moltenvk/*.mm)
  file(GLOB APPLE_HEADER_FILES LIST_DIRECTORIES false RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} moltenvk/*.h)
  list(APPEND SRC_FILES ${APPLE_SRC_FILES})
  list(APPEND HEADER_FILES ${APPLE_HEADER_FILES})
endif()

add_library(IGLVulkan ${SRC_FILES} ${HEADER_FILES})

target_link_libraries(IGLVulkan PRIVATE IGLLibrary)

igl_set_cxxstd(IGLVulkan 17)
igl_set_folder(IGLVulkan "IGL")

include(FetchContent)

# glslang
# cmake-format: off
#set(ENABLE_GLSLANG_BINARIES OFF CACHE BOOL "")
#set(ENABLE_HLSL             OFF CACHE BOOL "")
#set(ENABLE_CTEST            OFF CACHE BOOL "")
#set(ENABLE_OPT              OFF CACHE BOOL "")
#set(ENABLE_SPVREMAPPER      OFF CACHE BOOL "")
#set(SKIP_GLSLANG_INSTALL    ON  CACHE BOOL "")
#add_subdirectory(${IGL_ROOT_DIR}/third-party/deps/src/glslang "glslang")
#igl_set_folder(GenericCodeGen     "third-party/glslang")
#igl_set_folder(glslang            "third-party/glslang")
#igl_set_folder(MachineIndependent "third-party/glslang")
#igl_set_folder(OGLCompiler        "third-party/glslang")
#igl_set_folder(OSDependent        "third-party/glslang")
#igl_set_folder(SPIRV              "third-party/glslang")
#igl_set_folder(glslang-default-resource-limits "third-party/glslang")
# cmake-format: on
FetchContent_Declare(
    glslang
    GIT_REPOSITORY https://github.com/KhronosGroup/glslang.git
    GIT_TAG        vulkan-sdk-1.3.280.0
    GIT_SHALLOW    TRUE
    GIT_PROGRESS   TRUE
)
FetchContent_GetProperties(glslang)
string(TOLOWER "glslang" lc_glslang)
if (NOT ${lc_glslang}_POPULATED)
    FetchContent_Populate(glslang)
    option(SKIP_GLSLANG_INSTALL    "Skip installation"                       OFF)
    option(ENABLE_SPVREMAPPER      "Enables building of SPVRemapper"         OFF)
    option(ENABLE_GLSLANG_BINARIES "Builds glslang and spirv-remap"          OFF)
    option(ENABLE_HLSL             "Enables HLSL input support"              OFF)
    option(ENABLE_RTTI             "Enables RTTI"                            ON)
    option(ENABLE_EXCEPTIONS       "Enables Exceptions"                      ON)
    option(ENABLE_OPT              "Enables spirv-opt capability if present" ON)
    option(ENABLE_CTEST            "Enables testing"                         OFF)
    add_subdirectory(${${lc_glslang}_SOURCE_DIR} ${${lc_glslang}_BINARY_DIR})
endif ()

FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        10.2.1
    GIT_SHALLOW    TRUE
    GIT_PROGRESS   TRUE
)
FetchContent_MakeAvailable(fmt)

#target_link_libraries(IGLVulkan PUBLIC glslang SPIRV glslang-default-resource-limits)

find_package(Vulkan REQUIRED)

#target_link_libraries(IGLVulkan PUBLIC Vulkan::Vulkan)
#target_link_libraries(IGLVulkan PUBLIC fmt::fmt)
#target_include_directories(IGLVulkan PUBLIC "${IGL_ROOT_DIR}/third-party/deps/src/fmt/include")
#target_include_directories(IGLVulkan PUBLIC "${IGL_ROOT_DIR}/third-party/deps/src/volk")
FetchContent_Declare(
    volk
    GIT_REPOSITORY https://github.com/zeux/volk.git
    GIT_TAG        vulkan-sdk-1.3.280.0
    GIT_SHALLOW    TRUE
    GIT_PROGRESS   TRUE
)
FetchContent_MakeAvailable(volk)

#target_include_directories(IGLVulkan PUBLIC "${IGL_ROOT_DIR}/third-party/deps/src/vma/include")
FetchContent_Declare(
    vma
    GIT_REPOSITORY https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.git
    GIT_TAG        v3.0.1
    GIT_SHALLOW    TRUE
    GIT_PROGRESS   TRUE
)
FetchContent_MakeAvailable(vma)
#target_include_directories(IGLVulkan PUBLIC "${IGL_ROOT_DIR}/third-party/deps/src/SPIRV-Headers/include")
FetchContent_Declare(
    SPIRV-Headers
    GIT_REPOSITORY https://github.com/KhronosGroup/SPIRV-Headers.git
    GIT_TAG        vulkan-sdk-1.3.280.0
    GIT_SHALLOW    TRUE
    GIT_PROGRESS   TRUE
)
FetchContent_MakeAvailable(SPIRV-Headers)

target_link_libraries(IGLVulkan PUBLIC glslang)
target_link_libraries(IGLVulkan PUBLIC Vulkan::Vulkan)
target_link_libraries(IGLVulkan PUBLIC fmt::fmt)
target_link_libraries(IGLVulkan PUBLIC volk::volk_headers)
target_link_libraries(IGLVulkan PUBLIC VulkanMemoryAllocator)
target_link_libraries(IGLVulkan PUBLIC SPIRV-Headers::SPIRV-Headers)

if(WIN32)
  add_definitions("-DVK_USE_PLATFORM_WIN32_KHR=1")
  add_definitions("-DNOMINMAX")
endif()

if(UNIX AND NOT APPLE AND NOT ANDROID)
  add_definitions("-DVK_USE_PLATFORM_XLIB_KHR=1")
endif()

if(ANDROID)
  add_definitions("-DVK_USE_PLATFORM_ANDROID_KHR=1")
endif()

if(APPLE)
  add_definitions("-DVK_USE_PLATFORM_METAL_EXT=1")
  target_link_libraries(IGLVulkan PRIVATE "-framework QuartzCore")
  target_link_libraries(IGLVulkan PRIVATE "-framework Metal")
  if(MACOSX)
    target_link_libraries(IGLVulkan PRIVATE "-framework Cocoa")
  endif()
  if(IOS)
    target_link_libraries(IGLVulkan PRIVATE "-framework UIKit")
  endif()
endif()

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  target_compile_options(IGLVulkan PRIVATE "-Wno-nullability-completeness")
endif()
