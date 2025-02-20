cmake_minimum_required(VERSION 3.24.0)

# ----------------------------------------------------- config vcpkg

get_filename_component(XD_PROJECT_NAME ${CMAKE_CURRENT_SOURCE_DIR} NAME)
string(REPLACE " " "_" XD_PROJECT_NAME ${XD_PROJECT_NAME})
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/res/vcpkg.json.in" "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg.json")

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/.config.cmake")
    include(.config.cmake)
else()
    message(WARNING ${XD_PROJECT_NAME} "::CMakeLists::Warning: File ${CMAKE_CURRENT_SOURCE_DIR}/.config.cmake not found, Please create one.")
endif()

if (NOT DEFINED VCPKG_ROOT)
    message(FATAL_ERROR  ${XD_PROJECT_NAME} "::CMakeLists::Error: Vcpkg not found. Please define CMAKE_TOOLCHAIN_FILE var to point the dir to vcpkg.cmake")
else()
    set(CMAKE_TOOLCHAIN_FILE ${CMAKE_TOOLCHAIN_FILE} ${VCPKG_ROOT})
    message(STATUS ${XD_PROJECT_NAME} "::CMakeLists::Info: Vcpkg - found: ${VCPKG_ROOT}")
endif()

# ----------------------------------------------------- init proj

project(${XD_PROJECT_NAME})

add_subdirectory(3rd)

set(CMAKE_C_STANDARD 23)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED true)

# ----------------------------------------------------- collect src

if(NOT DEFINED CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE DEBUG)
endif()

if(NOT DEFINED XD_RENDER_IMPL)
set(XD_RENDER_IMPL "vk")
endif()

file(GLOB_RECURSE ALL_IMPL "core/src/*.ixx" "core/src/*.cxx" "core/renderImpl/${XD_RENDER_IMPL}/*.ixx" "core/renderImpl/${XD_RENDER_IMPL}/*.cxx")
file(GLOB_RECURSE ALL_TEMPLATE_INST "core/src/*.tpp")
set_source_files_properties(${ALL_IMPL} PROPERTIES LANGUAGE CXX)

file(GLOB_RECURSE ALL_SOURCE
    "core/src/*.c"
    "core/renderImpl/${XD_RENDER_IMPL}/*.c"

    "core/src/*.cpp"
    "core/renderImpl/${XD_RENDER_IMPL}/*.cpp"
)

include_directories(
    "${CMAKE_SOURCE_DIR}/core/src"
    "${CMAKE_SOURCE_DIR}/core/renderImpl"
    "${CMAKE_SOURCE_DIR}/3rd/stduuid/include"
    "${CMAKE_SOURCE_DIR}/3rd/mio/single_include"
)
add_executable(${PROJECT_NAME} ${ALL_SOURCE} ${ALL_IMPL} ${ALL_TEMPLATE_INST})

message(STATUS "\n-- " ${PROJECT_NAME} ":: # ----------------------------------------------------- build conf\n--")

# 控制台仅在 Debug 模式下生成
if(${CMAKE_BUILD_TYPE} MATCHES ".*(D|d)(E|e)(B|b)(U|u)(G|g).*")
message(STATUS "${PROJECT_NAME}::CMakeLists.txt::info: 将编译带有控制台的目标程序")
else()
message(STATUS "${PROJECT_NAME}::CMakeLists.txt::info: 将编译隐藏控制台的目标程序")
target_link_options(${PROJECT_NAME} PRIVATE -mwindows)
endif()

# ----------------------------------------------------- 3rd pkg

message(STATUS "\n-- " ${PROJECT_NAME} ":: # ----------------------------------------------------- 3rd pkg\n--")

# 添加 vulkan
find_package(Vulkan REQUIRED)
message(STATUS ${PROJECT_NAME} "::CMakeLists::info: include vulkan from \"" ${Vulkan_INCLUDE_DIRS} "\"")
message(STATUS ${PROJECT_NAME} "::CMakeLists::info: link vulkan lib from \"" ${Vulkan_LIBRARIES} "\"")
target_include_directories(${PROJECT_NAME} PRIVATE ${Vulkan_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${Vulkan_LIBRARIES})

# 添加 Physx
# PhysX does not have a find_package setup therefore we have to find it manually
include(SelectLibraryConfigurations)

find_library(PhysXPvdSDK_LIBRARY_RELEASE NAMES PhysXPvdSDK_64 PhysXPvdSDK_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}" NO_DEFAULT_PATH REQUIRED)
find_library(PhysXPvdSDK_LIBRARY_DEBUG NAMES PhysXPvdSDK_64 PhysXPvdSDK_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug" NO_DEFAULT_PATH REQUIRED)
select_library_configurations(PhysXPvdSDK)

find_library(PhysXExtensions_LIBRARY_RELEASE NAMES PhysXExtensions_64 PhysXExtensions_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}" NO_DEFAULT_PATH REQUIRED)
find_library(PhysXExtensions_LIBRARY_DEBUG NAMES PhysXExtensions_64 PhysXExtensions_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug" NO_DEFAULT_PATH REQUIRED)
select_library_configurations(PhysXExtensions)

find_library(PhysXFoundation_LIBRARY_RELEASE NAMES PhysXFoundation_64 PhysXFoundation_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}" NO_DEFAULT_PATH REQUIRED)
find_library(PhysXFoundation_LIBRARY_DEBUG NAMES PhysXFoundation_64 PhysXFoundation_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug" NO_DEFAULT_PATH REQUIRED)
select_library_configurations(PhysXFoundation)

find_library(PhysXCommon_LIBRARY_RELEASE NAMES PhysXCommon_64 PhysXCommon_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}" NO_DEFAULT_PATH REQUIRED)
find_library(PhysXCommon_LIBRARY_DEBUG NAMES PhysXCommon_64 PhysXCommon_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug" NO_DEFAULT_PATH REQUIRED)
select_library_configurations(PhysXCommon)

find_library(PhysX_LIBRARY_RELEASE NAMES PhysX_64 PhysX_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}" NO_DEFAULT_PATH REQUIRED)
find_library(PhysX_LIBRARY_DEBUG NAMES PhysX_64 PhysX_static_64 PATH_SUFFIXES lib PATHS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/debug" NO_DEFAULT_PATH REQUIRED)
select_library_configurations(PhysX)

if(WIN32 OR APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${PhysXPvdSDK_LIBRARY} ${PhysXExtensions_LIBRARY} ${PhysXFoundation_LIBRARY} ${PhysXCommon_LIBRARY} ${PhysX_LIBRARY})
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE -Wl,--start-group ${PhysXPvdSDK_LIBRARY} ${PhysXExtensions_LIBRARY} ${PhysXFoundation_LIBRARY} ${PhysXCommon_LIBRARY} ${PhysX_LIBRARY} -Wl,--end-group)
endif()
target_include_directories(${PROJECT_NAME} PRIVATE "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/include")
message(STATUS ${PROJECT_NAME} "::CMakeLists::info: include physx from \"" "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/include" "\"")

# 添加 SDL
target_include_directories(${PROJECT_NAME} PRIVATE 3rd/SDL3/include)
target_link_libraries(${PROJECT_NAME} PRIVATE SDL3-static)
message(STATUS ${PROJECT_NAME} "::CMakeLists::info: include SDL3 from \"3rd/SDL3\"")

# 添加 SDL_image
target_include_directories(${PROJECT_NAME} PRIVATE 3rd/SDL3_image/include)
target_link_libraries(${PROJECT_NAME} PRIVATE SDL3_image)
message(STATUS ${PROJECT_NAME} "::CMakeLists::info: include SDL3 from \"3rd/SDL3_image\"")

# 添加 assimp
find_package(assimp CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE assimp::assimp)

# 添加 rapidJson
find_package(RapidJSON CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE rapidjson)

# 添加 lz4
find_package(lz4 CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE lz4::lz4)

# 添加 glm
# 链接 glm
target_include_directories(${PROJECT_NAME} PUBLIC ./3rd/glm/glm)
target_link_libraries(${PROJECT_NAME} PUBLIC glm)

message(STATUS "\n-- " ${PROJECT_NAME} ":: # ----------------------------------------------------- cmake end\n--")