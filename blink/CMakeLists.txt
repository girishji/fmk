cmake_minimum_required(VERSION 3.20 FATAL_ERROR)

# Fetch SDK (must be before project)
include(FetchContent)
FetchContent_Declare(
    pico-sdk
    GIT_REPOSITORY https://github.com/RaspberryPi/pico-sdk.git
    GIT_TAG 2e6142b15b8a75c1227dd3edbe839193b2bf9041 # release 1.4.0
)
FetchContent_MakeAvailable(pico-sdk)
include(${pico-sdk_SOURCE_DIR}/pico_sdk_init.cmake)
include(${pico-sdk_SOURCE_DIR}/pico_sdk_version.cmake)

project(
    fmk
    VERSION 0.1.0
    DESCRIPTION "LED-blinking application for Pico using FreeRTOS"
    HOMEPAGE_URL https://github.com/
    LANGUAGES ASM C CXX
)

set(CMAKE_C_STANDARD 11 CACHE STRING "")
set(CMAKE_C_STANDARD_REQUIRED yes CACHE BOOL "")
set(CMAKE_CXX_STANDARD 17 CACHE STRING "")
set(CMAKE_CXX_STANDARD_REQUIRED yes CACHE BOOL "")
set(CMAKE_CXX_EXTENSIONS no CACHE BOOL "")

# Fetch FreeRTOS
FetchContent_Declare(
    free-rtos
    GIT_REPOSITORY https://github.com/FreeRTOS/FreeRTOS-Kernel
    GIT_TAG 13f034eb7480f11c9fc43e7b51a1e55656985bf4 # smp branch of v10.5.1
)
FetchContent_GetProperties(free-rtos)
if(NOT free-rtos_POPULATED)
    FetchContent_Populate(free-rtos)
    include(${free-rtos_SOURCE_DIR}/portable/ThirdParty/GCC/RP2040/FreeRTOS_Kernel_import.cmake)
    add_subdirectory(${free-rtos_SOURCE_DIR} ${free-rtos_BINARY_DIR})
endif()

# Initialize the SDK
pico_sdk_init()

add_executable(blink main.c)

# Add header search paths
target_include_directories(blink PRIVATE ${CMAKE_CURRENT_LIST_DIR})

# Pull in common dependencies
target_link_libraries(blink pico_stdlib FreeRTOS-Kernel FreeRTOS-Kernel-Heap4)

# Enable/disable STDIO via USB and UART for debugging
pico_enable_stdio_usb(blink 1)
# pico_enable_stdio_uart(blink 1)

# Create map, bin, hex, uf2 file etc.
pico_add_extra_outputs(blink)
