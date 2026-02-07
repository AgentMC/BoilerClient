# Generated Cmake Pico project file

cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Initialise pico_sdk from installed location
# (note this can come from environment, CMake cache etc)

# == DO NOT EDIT THE FOLLOWING LINES for the Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
    set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(sdkVersion 2.2.0)
set(toolchainVersion 14_2_Rel1)
set(picotoolVersion 2.2.0-a4)
set(picoVscode ${USERHOME}/.pico-sdk/cmake/pico-vscode.cmake)
if (EXISTS ${picoVscode})
    include(${picoVscode})
endif()
# ====================================================================================
set(PICO_BOARD pico_w CACHE STRING "Board type")

# Pull in Raspberry Pi Pico SDK (must be before project)
include(pico_sdk_import.cmake)

project(boiler_client C CXX ASM)

#DO NOT FORGET TO SET THESE!
set(WIFI_SSID "Your Wi-Fi SSID")
set(WIFI_PASSWORD "Your Wi-Fi Password")
set(TLS_CLIENT_SERVER "Your-Boiler-Manager-Server.com")

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

# Add OneWire library
add_subdirectory(Modules/pico-onewire)

# Add executable. Default name is the project name, version 0.1
add_executable(boiler_client_background
        boiler_client.cpp
        )
target_compile_definitions(boiler_client_background PRIVATE
        WIFI_SSID=\"${WIFI_SSID}\"
        WIFI_PASSWORD=\"${WIFI_PASSWORD}\"
        TLS_CLIENT_SERVER=\"${TLS_CLIENT_SERVER}\"
        )
target_compile_options(boiler_client_background PRIVATE -Wno-psabi) # Ignore warnings from stl_tree.h
target_include_directories(boiler_client_background PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}
        ${CMAKE_CURRENT_LIST_DIR}/.. # for our common lwipopts and mbedtls_config.h
        )
#Enable USB, disable UART
pico_enable_stdio_usb(boiler_client_background 1)
pico_enable_stdio_uart(boiler_client_background 0)
#dependencies
target_link_libraries(boiler_client_background
        pico_cyw43_arch_lwip_threadsafe_background
        pico_lwip_mbedtls
        pico_mbedtls
        pico_stdlib
        pico_one_wire
        )
#generate uf2
pico_add_extra_outputs(boiler_client_background)

# Ignore warnings from lwip code
set_source_files_properties(
        ${PICO_LWIP_PATH}/src/apps/altcp_tls/altcp_tls_mbedtls.c
        PROPERTIES
        COMPILE_OPTIONS "-Wno-unused-result"
        )