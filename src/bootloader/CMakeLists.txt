SET(CMAKE_CXX_CREATE_SHARED_LIBRARY "${CMAKE_LINKER} <LINK_FLAGS> -o <TARGET> <OBJECTS> <LINK_LIBRARIES>")
SET(COMMON_FLAGS "\
-Os				  \
-s				  \
-fno-stack-protector		 \
-ffreestanding				 \
-fshort-wchar				 \
-fPIC")

SET(COMMON_WARNING_FLAGS "-Wall \
-Werror=all \
-Wno-error=unused-function \
-Wno-error=unused-but-set-variable \
-Wno-error=unused-variable \
-Wno-error=deprecated-declarations \
-Wextra \
-Werror=frame-larger-than=65536 \
-Wno-unused-parameter \
-Wno-sign-compare \
-Wno-old-style-declaration")

SET(LDFLAGS "\
-nostdlib           \
-znocombreloc       \
-shared				\
-Bsymbolic          \
-T ${GNU_EFI_LIB}/elf_${ARCH}_efi.lds \
-L ${GNU_EFI_LIB}   \
${GNU_EFI_LIB}/crt0-efi-${ARCH}.o")

SET(CMAKE_C_FLAGS "${COMMON_FLAGS} ${BOARD_C_FLAGS} -std=gnu11")
SET(CMAKE_CXX_FLAGS "${COMMON_FLAGS} ${BOARD_CXX_FLAGS} -std=gnu++17")
SET(CMAKE_SHARED_LINKER_FLAGS ${LDFLAGS})

# - Add user project to CMake project
INCLUDE_DIRECTORIES(${GNU_EFI_INC} ${GNU_EFI_INC}/${ARCH} ${COMMON_INC})

ADD_DEFINITIONS(-DGNU_EFI_USE_MS_ABI -D_ELF -D__amd64__)

GET_PROPERTY(EFILOAD_SRC GLOBAL PROPERTY EFILOAD_SRC)
FILE(GLOB_RECURSE BOOTLOADER_SRC "${BOARD}/*.c" "${BOARD}/*.cpp")
FILE(GLOB_RECURSE BOOTLOADER_SRC_ASM "${BOARD}/*.S")

SET_PROPERTY(SOURCE ${BOOTLOADER_SRC_ASM} PROPERTY LANGUAGE C)
SET_SOURCE_FILES_PROPERTIES(${BOOTLOADER_SRC_ASM} PROPERTIES COMPILE_FLAGS "-x assembler-with-cpp -D_${ARCH}_ -D__amd64__")
ADD_LIBRARY(bootloader SHARED ${BOOTLOADER_SRC} ${EFILOAD_SRC} ${BOOTLOADER_SRC_ASM})

TARGET_LINK_LIBRARIES(bootloader efi gnuefi)
SET_TARGET_PROPERTIES(bootloader PROPERTIES 
									SUFFIX .so
									LINKER_LANGUAGE CXX)

# Build target

SET(OBJCOPYFLAGS -j.text -j.sdata -j.data -j.dynamic -j.dynsym  -j.rel -j.rela -j.reloc --target=efi-app-${ARCH})

ADD_CUSTOM_COMMAND(TARGET bootloader POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} ${OBJCOPYFLAGS} $<TARGET_FILE:bootloader> ${CMAKE_CURRENT_BINARY_DIR}/BOOTX64.EFI
		VERBATIM
        DEPENDS bootloader
        COMMENT "Generating .efi file ...")
