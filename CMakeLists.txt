cmake_minimum_required(VERSION 3.25)

project(RPLIDAR_SDK VERSION 2.1.0 LANGUAGES C CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set C standard
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Build configuration
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Paranoid build option
option(PARANOID_BUILD "Enable paranoid build with strict compiler checks" ON)

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Target platform: ${CMAKE_SYSTEM_NAME}")
message(STATUS "Target architecture: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "Paranoid build: ${PARANOID_BUILD}")

# Compiler flags setup
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Common compile definitions
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_definitions(_DEBUG DEBUG)
else()
    add_compile_definitions(NDEBUG _NDEBUG)
endif()

# Architecture specific definitions
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|amd64|AMD64")
    add_compile_definitions(_CPU_ENDIAN_SMALL)
endif()

# Platform specific settings
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    add_compile_definitions(_MACOS)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # Add rt library for Linux
    link_libraries(rt)
endif()

# Compiler specific flags
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    # Basic flags
    add_compile_options(
        -Wall
        -funsigned-char
    )

    # C specific flags
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wstrict-prototypes")

    # Paranoid build flags - only critical error detection
    if(PARANOID_BUILD)
        message(STATUS "Enabling paranoid build flags (critical errors only)...")
        add_compile_options(
            # Memory safety - critical errors
            -Werror=array-bounds
            -Werror=shift-overflow
            -Werror=stack-protector
            -Werror=format-security
            -Werror=null-dereference
            -Werror=uninitialized
            -Werror=maybe-uninitialized

            # Logic errors - critical
            -Werror=return-type
            -Werror=sequence-point
            -Werror=strict-aliasing
            -Werror=strict-overflow

            # Memory protection
            -fstack-protector-strong
            -D_FORTIFY_SOURCE=2
        )

        # Additional C++ specific critical checks
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror=delete-non-virtual-dtor")

        # Debug builds: enable sanitizers for critical runtime checks
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            add_compile_options(-fsanitize=address -fsanitize=undefined -fno-omit-frame-pointer)
            add_link_options(-fsanitize=address -fsanitize=undefined)
        endif()
    endif()

    # Debug flags
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-ggdb)
        if(PARANOID_BUILD)
            add_compile_options(-fsanitize=address -fsanitize=undefined -fno-omit-frame-pointer)
            add_link_options(-fsanitize=address -fsanitize=undefined)
        endif()
    else()
        add_compile_options(-O2)
        if(PARANOID_BUILD)
            add_compile_options(-D_FORTIFY_SOURCE=2)
        endif()
    endif()
endif()

# Add subdirectories
add_subdirectory(sdk)
add_subdirectory(app)

