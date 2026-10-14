cmake_minimum_required(VERSION 3.20)

set(mcpu_flags
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
)

project(SelfBalancingRobot C CXX ASM)
add_executable(${PROJECT_NAME}
    src/startup_stm32f411xx.s
    src/main.c
    src/syscall.c
)
target_compile_definitions(${PROJECT_NAME} PRIVATE
    STM32F411xE
)
target_compile_options(${PROJECT_NAME} PRIVATE
    ${mcpu_flags}
    -Wall
    -Wextra
    -Wpedantic
    -fdata-sections
    -ffunction-sections
)
target_link_options(${PROJECT_NAME} PRIVATE
    ${mcpu_flags}
    -T ${CMAKE_SOURCE_DIR}/src/stm32f411xe_flash.ld
    --specs=nano.specs
    --specs=nosys.specs
    -Wl,--gc-sections
)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}> ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${PROJECT_NAME}> ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}>
)