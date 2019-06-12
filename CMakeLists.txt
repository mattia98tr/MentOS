cmake_minimum_required(VERSION 2.8.4)

# ------------------------------------------------------------------------------

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DDEBUG_STDIO")

# ------------------------------------------------------------------------------
# Add the sub-directories.
add_subdirectory(mentos)
add_subdirectory(initscp)

# ------------------------------------------------------------------------------
# Generate the initrd filesystem.
add_custom_target(
    initfs
    COMMAND echo "---------------------------------------------"
    COMMAND echo "Initializing 'initfs'..."
    COMMAND echo "---------------------------------------------"
    COMMAND initfscp -s ${CMAKE_SOURCE_DIR}/files -t ${CMAKE_BINARY_DIR}/initfs -m /dev
    COMMAND echo "---------------------------------------------"
    COMMAND echo "Done!"
    COMMAND echo "---------------------------------------------"
    DEPENDS initfscp
)

# ------------------------------------------------------------------------------
# Set memory size.
SET(MEMORY_SIZE 1096M)

# ------------------------------------------------------------------------------
# Builds the code and runs qemu with the built Os.
SET(EMULATOR qemu-system-i386)
if (${DEBUGGING_TYPE} STREQUAL DEBUG_LOG)
    SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -serial file:serial.log)
elseif (${DEBUGGING_TYPE} STREQUAL DEBUG_STDIO)
    SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -serial stdio)
endif ()
SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -vga std)
SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -k it)
SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -sdl)
SET(EMULATOR_FLAGS ${EMULATOR_FLAGS} -m ${MEMORY_SIZE})
SET(EMULATOR_KERNEL -kernel mentos/kernel.bin)
SET(EMULATOR_FS -initrd initfs)

add_custom_target(
    qemu
    COMMAND ${EMULATOR} ${EMULATOR_FLAGS} ${EMULATOR_KERNEL} ${EMULATOR_FS}
    DEPENDS MentOs
    DEPENDS initfs
)

# ------------------------------------------------------------------------------
# Builds the code and runs qemu with the built Os.
SET(EMULATOR_FLAGS_GDB ${EMULATOR_FLAGS} -s -S)
SET(GDB_PROG cgdb)
SET(GDB_ARGS -x ${CMAKE_SOURCE_DIR}/.gdb.debug)

add_custom_target(
    qemu-gdb
    COMMAND xterm -geometry 160x30-0-0 -e ${EMULATOR} ${EMULATOR_FLAGS_GDB} ${EMULATOR_KERNEL} ${EMULATOR_FS} &
    COMMAND xterm -geometry 160x30-0+0 -e ${GDB_PROG} ${GDB_ARGS} &
    DEPENDS MentOs
    DEPENDS initfs
)

