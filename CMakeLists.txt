cmake_minimum_required(VERSION 3.5.0)
project(stm32_blink C CXX ASM)

IF(NOT CMAKE_CROSSCOMPILING)
    message(FATAL_ERROR "Cross compiling only. Please use -DCMAKE_TOOLCHAIN_FILE=/PATH/TO/TOOLCHAIN_FILE .")
ENDIF(NOT CMAKE_CROSSCOMPILING)

add_definitions(-DSTM32F0 -DPRINTF_DISABLE_SUPPORT_FLOAT)

set(LIBOPENCM3_DIR ${CMAKE_SOURCE_DIR}/libopencm3)
add_custom_target(libopencm3 make WORKING_DIRECTORY ${LIBOPENCM3_DIR})
link_directories(${LIBOPENCM3_DIR}/lib)

file(GLOB_RECURSE USER_SOURCES src/*.c *.h src/*.cpp)

include_directories(
        ${LIBOPENCM3_DIR}/include
        inc)

set(STM32F0_FLAGS "-mcpu=cortex-m0 -mthumb -mthumb-interwork -msoft-float")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall ${STM32F0_FLAGS} ")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -Wall --std=c++14 ${STM32F0_FLAGS}")
#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${CMAKE_SOURCE_DIR}/libopencm3.ld -nostartfiles --specs=rdimon.specs    ")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${CMAKE_SOURCE_DIR}/libopencm3.ld -nostartfiles --specs=nano.specs --specs=nosys.specs -lc -Wl,-Map=${PROJECT_SOURCE_DIR}/build/${PROJECT_NAME}.map")


function(add_bin_from_elf bin elf)
    add_custom_target(${bin}
            ALL ${CMAKE_OBJCOPY}
            -Obinary ${elf} ${bin} DEPENDS ${elf})
endfunction(add_bin_from_elf)

add_subdirectory(src)