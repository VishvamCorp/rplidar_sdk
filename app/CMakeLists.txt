# Application examples for RPLIDAR SDK

# Function to create an application target
function(add_rplidar_app APP_NAME)
    add_executable(${APP_NAME} ${APP_NAME}/main.cpp)

    # Link with SDK library
    target_link_libraries(${APP_NAME} PRIVATE sl_lidar_sdk)

    # Include directories
    target_include_directories(${APP_NAME} PRIVATE
        ../sdk/include
        ../sdk/src
    )

    # Platform specific libraries
    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        target_link_libraries(${APP_NAME} PRIVATE pthread rt)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
        target_link_libraries(${APP_NAME} PRIVATE pthread)
    endif()

    # Set output properties
    set_target_properties(${APP_NAME} PROPERTIES
        LANGUAGE CXX
        CXX_STANDARD 11
    )

    # Paranoid build settings for applications
    if(PARANOID_BUILD)
        # Only add critical runtime checks for debug builds
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            target_compile_definitions(${APP_NAME} PRIVATE
                _GLIBCXX_ASSERTIONS
            )
        endif()
        message(STATUS "App ${APP_NAME}: Paranoid build enabled with critical checks")
    endif()
endfunction()

# Create application targets
add_rplidar_app(simple_grabber)
add_rplidar_app(ultra_simple)
add_rplidar_app(custom_baudrate)

# Install applications
install(TARGETS simple_grabber ultra_simple custom_baudrate
    RUNTIME DESTINATION bin
)
