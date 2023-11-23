cmake_minimum_required(VERSION 3.22)

# Include gcc-arm-none-eabi
include(cmake/gcc-arm-none-eabi.cmake)

# Setup compiler settings
set(CMAKE_C_STANDARD                11)
set(CMAKE_C_STANDARD_REQUIRED       ON)
set(CMAKE_C_EXTENSIONS              ON)
set(CMAKE_CXX_STANDARD              20)
set(CMAKE_CXX_STANDARD_REQUIRED     ON)
set(CMAKE_CXX_EXTENSIONS            ON)

# Project name
project(imu C CXX ASM)

# Print info of project
set(PROJ_PATH  ${CMAKE_CURRENT_SOURCE_DIR})
message("Project name: " ${CMAKE_PROJECT_NAME})
message("Project dir:" ${CMAKE_CURRENT_SOURCE_DIR})

# Set MCU options
set(CPU "-mcpu=cortex-m4")
set(FPU "-mfpu=fpv4-sp-d16")
set(FLOAT_ABI "-mfloat-abi=hard")
set(MCU_FLAGS ${CPU} -mthumb ${FPU} ${FLOAT_ABI})

# Set optimization
set(OPT_FLAGS "-Og")

# Set build type (debug/release)
set(DEBUG 0)
if (DEBUG)
  set(OPT_FLAGS ${OPT_FLAGS} -g3 -ggdb)
  set(CMAKE_BUILD_TYPE Debug)
else()
  set(CMAKE_BUILD_TYPE Release)
endif()

message("Build type: "  ${CMAKE_BUILD_TYPE})

# Set linker script
set(LINKER_SCRIPT       ${PROJ_PATH}/STM32G431CBUx_FLASH.ld)
set(EXECUTABLE          ${CMAKE_PROJECT_NAME})

# Set source files
set(SOURCE_FILES
 # TODO: add source files
)

# Set asm sources
set(ASM_SOURCES
  ${PROJ_PATH}/startup_stm32g431xx.s
)
# Set include directories
set(INCLUDE_DIRS
  # TODO: add include directories
)

# Set definition
set(SYS_DEFS
  -DUSE_HAL_DRIVER
  -DSTM32G431xx
)

# Executable files
add_executable(${EXECUTABLE} 
  ${SOURCE_FILES}
  ${ASM_SOURCES}
)

# Include paths
target_include_directories(${EXECUTABLE} PRIVATE ${INCLUDE_DIRS})

# Project symbols
target_compile_definitions(${EXECUTABLE} PRIVATE ${SYS_DEFS})

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
  ${MCU_FLAGS}
  -Wall
  -Wextra
  -Wpedantic
  -Wno-unused-parameter
  ${OPT_FLAGS}
)

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
  ${MCU_FLAGS}
  -T${LINKER_SCRIPT}
  -Wl,-Map=${CMAKE_PROJECT_NAME}.map
  --specs=nosys.specs
  -u _printf_float                # STDIO float formatting support
  -Wl,--start-group
  -lc
  -lm
  -lstdc++
  -lsupc++
  -Wl,--end-group
  -Wl,--print-memory-usage
)

# Execute post-build to print size
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
  COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>
)

# Convert output to hex and binary
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
  COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.hex
)

# Convert to bin file -> add conditional check?
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
  COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.bin
)