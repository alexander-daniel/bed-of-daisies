cmake_minimum_required(VERSION 3.20)

project("test-plugin" VERSION 0.0.1)
set(FIRMWARE_NAME ${CMAKE_PROJECT_NAME})

add_executable(${FIRMWARE_NAME})

target_sources(${FIRMWARE_NAME}
  PRIVATE
  ${CMAKE_CURRENT_LIST_DIR}/src/main.cpp
)

set(LIBDAISY_DIR ${CMAKE_CURRENT_LIST_DIR}/lib/libDaisy)
add_subdirectory(${LIBDAISY_DIR} libdaisy)
set(DAISYSP_DIR ${CMAKE_CURRENT_LIST_DIR}/lib/DaisySP)
add_subdirectory(${DAISYSP_DIR} DaisySP)

target_link_libraries(${FIRMWARE_NAME}
  PRIVATE
  daisy
  DaisySP
)

set(LINKER_SCRIPT ${LIBDAISY_DIR}/core/STM32H750IB_flash.lds)

set_target_properties(${FIRMWARE_NAME} PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    SUFFIX ".elf"
    LINK_DEPENDS ${LINKER_SCRIPT}
)

target_link_options(${FIRMWARE_NAME} PUBLIC
    -T ${LINKER_SCRIPT}
    -Wl,-Map=${FIRMWARE_NAME}.map,--cref
    -Wl,--check-sections
    -Wl,--unresolved-symbols=report-all
    -Wl,--warn-common
    -Wl,--warn-section-align
    -Wl,--print-memory-usage
)

add_custom_command(TARGET ${FIRMWARE_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY}
    ARGS -O ihex
    -S ${FIRMWARE_NAME}.elf
    ${FIRMWARE_NAME}.hex
    BYPRODUCTS
    ${FIRMWARE_NAME}.hex
    COMMENT "Generating HEX image"
VERBATIM)

add_custom_command(TARGET ${FIRMWARE_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY}
    ARGS -O binary
    -S ${FIRMWARE_NAME}.elf
    ${FIRMWARE_NAME}.bin
    BYPRODUCTS
    ${FIRMWARE_NAME}.bin
    COMMENT "Generating binary image"
VERBATIM)

add_custom_command(TARGET ${FIRMWARE_NAME} POST_BUILD
    COMMAND mkdir -p ${CMAKE_CURRENT_LIST_DIR}/build/artifacts/ 
    && mv ${FIRMWARE_NAME}.* ${CMAKE_CURRENT_LIST_DIR}/build/artifacts/
    COMMENT "Moving artifacts to build/output/"
)
