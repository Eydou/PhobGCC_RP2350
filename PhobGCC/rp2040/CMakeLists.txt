cmake_minimum_required(VERSION 3.12)

set(PICO_SDK_PATH "" CACHE PATH "Path to the Raspberry Pi Pico SDK")

if(NOT PICO_SDK_PATH)
	if(DEFINED ENV{PICO_SDK_PATH} AND NOT "$ENV{PICO_SDK_PATH}" STREQUAL "")
		set(PICO_SDK_PATH "$ENV{PICO_SDK_PATH}")
	elseif(EXISTS "${CMAKE_CURRENT_LIST_DIR}/pico-sdk/pico_sdk_init.cmake")
		set(PICO_SDK_PATH "${CMAKE_CURRENT_LIST_DIR}/pico-sdk")
	elseif(EXISTS "${CMAKE_CURRENT_LIST_DIR}/../../pico-sdk/pico_sdk_init.cmake")
		set(PICO_SDK_PATH "${CMAKE_CURRENT_LIST_DIR}/../../pico-sdk")
	elseif(EXISTS "${CMAKE_CURRENT_LIST_DIR}/../../../pico-sdk/pico_sdk_init.cmake")
		set(PICO_SDK_PATH "${CMAKE_CURRENT_LIST_DIR}/../../../pico-sdk")
	endif()
endif()

if(NOT EXISTS "${PICO_SDK_PATH}/pico_sdk_init.cmake")
	message(FATAL_ERROR
		"Could not find the Raspberry Pi Pico SDK.\n"
		"Set PICO_SDK_PATH in the environment or pass -DPICO_SDK_PATH=/absolute/path/to/pico-sdk when running CMake."
	)
endif()

include("${PICO_SDK_PATH}/pico_sdk_init.cmake")

project(phobgcc_pico CXX C ASM)

pico_sdk_init()

set(PHOB_PICO_TARGET_NAME phobgcc_pico)
if(PICO_PLATFORM MATCHES "^rp2350")
	set(PHOB_PICO_TARGET_NAME phobgcc_rp2350)
elseif(PICO_PLATFORM MATCHES "^rp2040")
	set(PHOB_PICO_TARGET_NAME phobgcc_rp2040)
endif()

include_directories(
	include
	../common
	../extras
	)

add_executable(${PHOB_PICO_TARGET_NAME}
	src/cvideo.cpp
	src/drawGraphics.cpp
	src/drawImage.cpp
	src/drawMenu.cpp
	src/graph.cpp
	src/joybus.cpp
	src/main.cpp
	src/navigateMenu.cpp
	src/storage.cpp
	)

pico_generate_pio_header(${PHOB_PICO_TARGET_NAME} ${CMAKE_CURRENT_LIST_DIR}/pio/joybus.pio)
pico_generate_pio_header(${PHOB_PICO_TARGET_NAME} ${CMAKE_CURRENT_LIST_DIR}/pio/cvideo.pio)

add_custom_command(OUTPUT ${CMAKE_CURRENT_LIST_DIR}/generated/joybus.pio.h
	DEPENDS ${CMAKE_CURRENT_LIST_DIR}/joybus.pio
	COMMAND Pioasm ${CMAKE_CURRENT_LIST_DIR}/joybus.pio ${CMAKE_CURRENT_LIST_DIR}/generated/joybus.pio.h
	)

add_custom_command(OUTPUT ${CMAKE_CURRENT_LIST_DIR}/generated/cvideo.pio.h
	DEPENDS ${CMAKE_CURRENT_LIST_DIR}/cvideo.pio
	COMMAND Pioasm ${CMAKE_CURRENT_LIST_DIR}/cvideo.pio ${CMAKE_CURRENT_LIST_DIR}/generated/cvideo.pio.h
	)

target_compile_definitions(${PHOB_PICO_TARGET_NAME}
	PRIVATE
	PICO_XOSC_STARTUP_DELAY_MULTIPLIER=64
	#PICO_DEFAULT_BOOT_STAGE2=boot2_generic_03h
	#PICO_FLASH_SPI_CLKDIV=8
	)

target_link_libraries(${PHOB_PICO_TARGET_NAME}
	PRIVATE
	pico_stdlib
	pico_bootrom
	pico_multicore
	pico_platform
	hardware_adc
	hardware_dma
	hardware_flash
	hardware_pio
	hardware_pwm
	hardware_spi
	hardware_sync
	hardware_timer)

pico_add_extra_outputs(${PHOB_PICO_TARGET_NAME})
pico_enable_stdio_usb(${PHOB_PICO_TARGET_NAME} 1)
