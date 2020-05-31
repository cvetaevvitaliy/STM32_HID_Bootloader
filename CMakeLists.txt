SET(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.7)

# specify cross compilers and tools
SET(CMAKE_C_COMPILER_WORKS 1)
SET(CMAKE_C_COMPILER arm-none-eabi-gcc)
SET(CMAKE_CXX_COMPILER_WORKS 1)
SET(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER  arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)

SET(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/STM32F103C8T6.ld)
set(DEVICE STM32F10X_MD)

SET(COMMON_FLAGS "-mcpu=cortex-m3 -mthumb -Wall -Os -std=gnu99 -fno-common -static -specs=nano.specs -specs=nosys.specs \
-Wextra -Wshadow -Wno-unused-variable -Wimplicit-function-declaration -Wredundant-decls -Wstrict-prototypes -Wmissing-prototypes \
-D${DEVICE} -nostdlib")

SET(CMAKE_CXX_FLAGS_INIT "${COMMON_FLAGS} -std=c++11")
SET(CMAKE_C_FLAGS_INIT "${COMMON_FLAGS} -std=gnu99")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-Wl,-gc-sections,--print-memory-usage -T ${LINKER_SCRIPT}")

PROJECT(usb_hid_bootloader_f1 C CXX ASM)
set(CMAKE_CXX_STANDARD 11)

# The default Flash Page size (Sector size) for LOW and MEDIUM STM32F103 devices is 1024 bytes
# High Density STM32F103 devices have 2 kB Flash Page size
SET(VECTOR_TABLE_OFFSET 0x0000)
SET(PAGE_SIZE 1024)

add_definitions(-D__weak=__attribute__\(\(weak\)\)
        -D__packed=__attribute__\(\(__packed__\)\)
        -DUSE_HAL_DRIVER -DSTM32F10X_MD
        -DVECTOR_TABLE_OFFSET=${VECTOR_TABLE_OFFSET}
        -DPAGE_SIZE=${PAGE_SIZE}
        -DTARGET_GENERIC_F103_PC13)


file(GLOB_RECURSE SOURCES "Inc/*.*"
        "Src/*.*"
        "CMSIS/Device/ST/STM32F10x/Source*.*")

include_directories(Inc CMSIS/Device/ST/STM32F10x/Include CMSIS/Include/)

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

set(CMAKE_EXE_LINKER_FLAGS
        "${CMAKE_EXE_LINKER_FLAGS} -Wl,-nostdlib,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map")

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")