cmake_minimum_required(VERSION 3.31)
project(Vulkan-Media-Player C)

set(CMAKE_C_STANDARD 23)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_FLAGS_DEBUG "-std=c2x -g -O0 -fno-omit-frame-pointer -DDEBUG")
set(CMAKE_C_FLAGS_RELEASE "-std=c2x -O2 -DNDEBUG -flto -march=native -fdata-sections -ffunction-sections")


# Fetch required libraries for building.
include(FetchContent)

FetchContent_Declare(
        cglm
        GIT_REPOSITORY https://github.com/recp/cglm.git
        GIT_TAG v0.9.6
)
FetchContent_MakeAvailable(cglm)

FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw.git
        GIT_TAG 3.4
)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(glfw)


# Platform agnostic include and source files.
include_directories(
        ${CMAKE_SOURCE_DIR}/src
        ${CMAKE_SOURCE_DIR}/libs
        ${CMAKE_SOURCE_DIR}/libs/glad/include
)

set(SOURCE_FILES
        libs/stb_image/stb_image.c
        libs/glad/src/gl.c
        src/entrypoint.c
        src/game/game.c
)


# Windows specific source files, includes and libraries.
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    message("Building for Windows")
    include_directories(src/platform/windows)

    set(SOURCE_FILES ${SOURCE_FILES}
            src/engine/platform/windows/windows_entrypoint.c
    )

    add_executable(${PROJECT_NAME} WIN32 ${SOURCE_FILES})
    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -Wpedantic -Werror
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE
            user32
            kernel32
            cglm
            glfw
    )
endif()


# Linux specific source files, includes and libraries.
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message("Building for Linux")
    include_directories(src/platform/linux)

    set(SOURCE_FILES ${SOURCE_FILES}
            src/engine/platform/linux/linux_entrypoint.c
    )

    add_executable(${PROJECT_NAME} ${SOURCE_FILES})
    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -Wpedantic -Werror
            $<$<CONFIG:Debug>: -fsanitize=undefined,address>
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE
            cglm
            glfw
    )
endif()


# MacOS specific source files, includes and libraries.
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    message("Building for MacOS")
    include_directories(src/platform/macos)

    set(SOURCE_FILES ${SOURCE_FILES}
            src/engine/platform/macos/macos_entrypoint.c
    )

    add_executable(${PROJECT_NAME} ${SOURCE_FILES})
    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -Wpedantic -Werror
            $<$<CONFIG:Debug>: -fsanitize=undefined,address>
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE
            cglm
            glfw
    )
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME "Vulkan Media Player")
