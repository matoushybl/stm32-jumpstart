cmake_minimum_required(VERSION 3.5.0)
project(stm32_blink C CXX ASM)

set(CMAKE_VERBOSE_MAKEFILE TRUE)

IF(NOT CMAKE_CROSSCOMPILING)
    message(FATAL_ERROR "Cross compiling only. Please use -DCMAKE_TOOLCHAIN_FILE=/PATH/TO/TOOLCHAIN_FILE .")
ENDIF(NOT CMAKE_CROSSCOMPILING)

add_definitions(-DSTM32F1)

file(GLOB_RECURSE USER_SOURCES src/*.c *.h)

set(LIBOPENCM3_DIR ${CMAKE_SOURCE_DIR}/libopencm3)

include_directories(
        ${LIBOPENCM3_DIR}/include
        inc)

add_custom_target(libopencm3 make WORKING_DIRECTORY ${LIBOPENCM3_DIR})

link_directories(${LIBOPENCM3_DIR}/lib)

set(STM32F1_FLAGS "-fno-common -ffunction-sections -fdata-sections")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall ${STM32F1_FLAGS} -mcpu=cortex-m3 -mthumb -mthumb-interwork -msoft-float")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} ${STM32F1_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${CMAKE_SOURCE_DIR}/libopencm3.ld -nostartfiles --specs=rdimon.specs   -Wl,-Map=${PROJECT_SOURCE_DIR}/build/${PROJECT_NAME}.map ")

function(add_bin_from_elf bin elf)
    add_custom_target(${bin}
            ALL ${CMAKE_OBJCOPY}
            -Obinary ${elf} ${bin} DEPENDS ${elf})
endfunction(add_bin_from_elf)

add_subdirectory(src)