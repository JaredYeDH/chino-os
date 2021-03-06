CMAKE_MINIMUM_REQUIRED(VERSION 3.0)
PROJECT(chino)

IF( NOT CMAKE_BUILD_TYPE )
    SET( CMAKE_BUILD_TYPE Debug)
ELSE()
    IF ((NOT CMAKE_BUILD_TYPE STREQUAL "Debug") AND (NOT CMAKE_BUILD_TYPE STREQUAL "Release"))
        MESSAGE(FATAL_ERROR "CMAKE_BUILD_TYPE must either be Debug or Release instead of ${CMAKE_BUILD_TYPE}")
    ENDIF()
ENDIF()

SET_PROPERTY(GLOBAL PROPERTY TARGET_SUPPORTS_SHARED_LIBS TRUE)
SET(COMMON_INC ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Add subproject
ADD_SUBDIRECTORY(thirdparty)
ADD_SUBDIRECTORY(src)

SET(FIRMWARE_DIR ${CMAKE_CURRENT_BINARY_DIR}/iso)

ADD_CUSTOM_COMMAND(OUTPUT firmware.iso
		COMMAND rm -f firmware.iso
        #COMMAND rm -rf ${FIRMWARE_DIR} && mkdir ${FIRMWARE_DIR} && mkdir ${FIRMWARE_DIR}/BOOT && mkdir ${FIRMWARE_DIR}/BOOT/EFI
        #COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/src/bootloader/BOOTX64.EFI ${FIRMWARE_DIR}/BOOT/EFI
		COMMAND rm -rf ${FIRMWARE_DIR} && mkdir ${FIRMWARE_DIR}
        COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/src/bootloader/BOOTX64.EFI ${FIRMWARE_DIR}
		COMMAND mkdir ${FIRMWARE_DIR}/chino && mkdir ${FIRMWARE_DIR}/chino/system
        COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/src/kernel/kernel ${FIRMWARE_DIR}/chino/system
		COMMAND xorriso -as mkisofs -R -f -no-emul-boot -o firmware.iso iso
        DEPENDS bootloader kernel
        COMMENT "Generating firmware.iso ...")

ADD_CUSTOM_COMMAND(OUTPUT kernel.hex
		COMMAND rm -f kernel.hex
		COMMAND ${CMAKE_ARCH_OBJCOPY} -O ihex ${CMAKE_CURRENT_BINARY_DIR}/src/kernel/kernel kernel.hex
        DEPENDS kernel
        COMMENT "Generating kernel.hex ...")

# TODO: add flash command

ADD_CUSTOM_TARGET(firmware DEPENDS firmware.iso)
ADD_CUSTOM_TARGET(hex DEPENDS kernel.hex)