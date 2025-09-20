# SDK Library for RPLIDAR
set(SDK_LIB_NAME sl_lidar_sdk)

# Collect all source files
set(SDK_SOURCES
    # Core library files
    src/sl_lidar_driver.cpp
    src/hal/thread.cpp
    src/sl_crc.cpp
    src/sl_serial_channel.cpp
    src/sl_lidarprotocol_codec.cpp
    src/sl_async_transceiver.cpp
    src/sl_tcp_channel.cpp
    src/sl_udp_channel.cpp

    # Data unpacker files
    src/dataunpacker/dataunpacker.cpp
    src/dataunpacker/unpacker/handler_hqnode.cpp
    src/dataunpacker/unpacker/handler_capsules.cpp
    src/dataunpacker/unpacker/handler_normalnode.cpp
)

# Platform specific source files
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    list(APPEND SDK_SOURCES
        src/arch/linux/net_serial.cpp
        src/arch/linux/net_socket.cpp
        src/arch/linux/timer.cpp
    )
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    list(APPEND SDK_SOURCES
        src/arch/macOS/net_serial.cpp
        src/arch/macOS/net_socket.cpp
        src/arch/macOS/timer.cpp
    )
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    list(APPEND SDK_SOURCES
        src/arch/win32/net_serial.cpp
        src/arch/win32/net_socket.cpp
        src/arch/win32/timer.cpp
    )
endif()

# Create static library
add_library(${SDK_LIB_NAME} STATIC ${SDK_SOURCES})

# Set library properties
set_target_properties(${SDK_LIB_NAME} PROPERTIES
    VERSION 1.12.0
    SOVERSION 1
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE ON
)

# Include directories
target_include_directories(${SDK_LIB_NAME}
    PUBLIC
        include
    PRIVATE
        src
        src/dataunpacker
        src/dataunpacker/unpacker
)

# Platform specific link libraries
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    target_link_libraries(${SDK_LIB_NAME} PRIVATE pthread rt)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    target_link_libraries(${SDK_LIB_NAME} PRIVATE pthread)
endif()

# Paranoid build specific settings
if(PARANOID_BUILD)
    # Add critical runtime checks only for debug builds
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_compile_definitions(${SDK_LIB_NAME} PRIVATE
            _GLIBCXX_ASSERTIONS
        )
    endif()

    message(STATUS "SDK: Paranoid build enabled with critical checks only")
endif()

# Install targets
install(TARGETS ${SDK_LIB_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY include/
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)
