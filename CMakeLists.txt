cmake_minimum_required(VERSION 3.13)

# Needs to be done before including SDK pico
# https://forums.raspberrypi.com/viewtopic.php?t=376508
#set(PICO_BOARD pimoroni_pico_plus2_rp2350)
#set(PICO_PLATFORM rp2350)
set(PICO_BOARD pico_w)
set(PICO_PLATFORM rp2040)

# Include the Pico SDK initialization script
include($ENV{PICO_SDK_PATH}/pico_sdk_init.cmake)
#include($ENV{PICO_SDK_PATH}/../pimoroni-pico/pimoroni_pico_import.cmake)
include($ENV{PICO_EXTRAS_PATH}/external/pico_extras_import.cmake)

project(pico_sound_mixer C CXX)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# Initialize the Pico SDK
pico_sdk_init()

# add all sounds files to executable
file(GLOB sounds_SRC
     "./sounds/*.c"
)

add_executable(${PROJECT_NAME}
        main.c
        ${sounds_SRC}
        )

target_compile_definitions(${PROJECT_NAME} PRIVATE
        # compile time configuration of I2S
        PICO_AUDIO_I2S_MONO_INPUT=1
        USE_AUDIO_I2S=1
        )

# Link the Pico SDK to your project
set(my_lib
     pico_stdlib
     pico_audio_i2s
     pico_rand
)

target_link_libraries(${PROJECT_NAME} ${my_lib})


# enable usb output, disable uart output
pico_enable_stdio_usb(${PROJECT_NAME} 1)
pico_enable_stdio_uart(${PROJECT_NAME} 0)

# create map/bin/hex file etc.
pico_add_extra_outputs(${PROJECT_NAME})
