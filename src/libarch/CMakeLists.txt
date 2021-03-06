SET(COMMON_FLAGS "\
-Os			\
-fno-common \
-fexceptions \
-ffunction-sections \
-fdata-sections \
-fstrict-volatile-bitfields \
-fno-stack-protector		 \
-fPIC						 \
-ekernel_entry")

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
-nostartfiles           \
-pie					\
-Wl,-static")

SET(CMAKE_C_FLAGS "${COMMON_FLAGS} -std=gnu11")
SET(CMAKE_CXX_FLAGS "${COMMON_FLAGS} -std=gnu++17")
SET(CMAKE_EXE_LINKER_FLAGS "${LDFLAGS}")

# - Add user project to CMake project
INCLUDE_DIRECTORIES(${GNU_EFI_INC} ${GNU_EFI_INC}/${ARCH} ${COMMON_INC})

FILE(GLOB_RECURSE ARCH_SRC "${ARCH}/*.c" "${ARCH}/*.cpp")
FILE(GLOB_RECURSE ARCH_SRC_ASM "${ARCH}/*.S")

SET_PROPERTY(SOURCE ${ARCH_SRC_ASM} PROPERTY LANGUAGE C)
SET_SOURCE_FILES_PROPERTIES(${ARCH_SRC_ASM} PROPERTIES COMPILE_FLAGS "-x assembler-with-cpp -D${ARCH}")

ADD_LIBRARY(arch STATIC ${ARCH_SRC} ${ARCH_SRC_ASM})

TARGET_LINK_LIBRARIES(arch)
SET_TARGET_PROPERTIES(arch PROPERTIES
							LINKER_LANGUAGE CXX)