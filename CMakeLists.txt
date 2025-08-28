cmake_minimum_required(VERSION 3.28)

include(pico_sdk_import.cmake)

project(pio_irq C CXX ASM)

set(CMAKE_CXX_STANDARD 26)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmodules-ts -fcommon")

pico_sdk_init()

add_library(pio_irq)
target_sources(pio_irq
        PUBLIC
)
target_sources(pio_irq
        PUBLIC
        FILE_SET cxx_modules TYPE CXX_MODULES FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/source/pio_irq.cpp
)

target_link_libraries( pio_irq
        # pico_stdlib
        $<COMPILE_ONLY:hardware_pio>
)

