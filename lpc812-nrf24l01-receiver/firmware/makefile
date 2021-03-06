.DEFAULT_GOAL := all


###############################################################################
# Configuration options for our application
TARGET := receiver
SOURCE_DIRS := .
BUILD_DIR := build

SYSTEM_CLOCK := 12000000

SOURCES := $(foreach sdir, $(SOURCE_DIRS), $(wildcard $(sdir)/*.c))
DEPENDENCIES := makefile receiver.ld platform.h
DEPENDENCIES += uart0.h rc_receiver.h rf.h spi.h persistent_storage.h
LIBS := gcc
LINKER_SCRIPT := receiver.ld


###############################################################################
# Pretty-print setup
V ?= $(VERBOSE)
ifneq ($(V), 1)
QUIET := @
ECHO := @echo
else
QUIET :=
ECHO := @true
endif


###############################################################################
# Toolchain setup
TOOLCHAIN_PREFIX := arm-none-eabi-
TOOLCHAIN_PATH :=

CC := $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)gcc
LD := $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)gcc
OBJCOPY := $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)objcopy

MKDIR_P = mkdir -p
FLASH_TOOL := lpc81x_isp.py --wait --run --flash
TERMINAL_PROGRAM := miniterm.py -p /dev/ttyUSB0 -b 115200 --echo


# FIXME: make sure we can do without that tool!
MAP_SUMMARY_TOOL := parse_gcc_map_file


###############################################################################
# Target and object file setup
OBJECTS := $(patsubst %.c, $(BUILD_DIR)/%.o, $(SOURCES))

TARGET_ELF := $(addprefix $(BUILD_DIR)/, $(TARGET).elf)
TARGET_BIN := $(addprefix $(BUILD_DIR)/, $(TARGET).bin)
TARGET_HEX := $(addprefix $(BUILD_DIR)/, $(TARGET).hex)
TARGET_MAP := $(addprefix $(BUILD_DIR)/, $(TARGET).map)

$(OBJECTS): $(DEPENDENCIES)
$(TARGET_MAP): $(TARGET_ELF)


###############################################################################
# Compiler and linker flags
CPU_FLAGS = -mthumb -mcpu=cortex-m0plus -mlittle-endian

CFLAGS := $(CPU_FLAGS)
CFLAGS += -std=c99
CFLAGS += -W -Wall -Wextra -Wpedantic
CFLAGS += -Wstrict-prototypes -Wshadow -Wwrite-strings
CFLAGS += -Wdeclaration-after-statement -Waddress -Wlogical-op
CFLAGS += -Wold-style-definition -Wmissing-prototypes -Wmissing-declarations
CFLAGS += -Wmissing-field-initializers -Wdouble-promotion -Wfloat-equal
CFLAGS += -Wswitch-enum -Wswitch-default -Wuninitialized -Wunknown-pragmas
CFLAGS += -Wundef
CFLAGS += -I. -isystem./LPC8xx
CFLAGS += -fsigned-char -fdata-sections -ffunction-sections -fno-common
CFLAGS += -fpack-struct=4
CFLAGS += -Os
CFLAGS += -D__SYSTEM_CLOCK=$(SYSTEM_CLOCK)

CFLAGS += -DNO_DEBUG
CFLAGS += -DBAUDRATE=38400
CFLAGS += -DENABLE_PREPROCESSOR_OUTPUT
#CLFAGS += -DEXTENDED_PREPROCESSOR_OUTPUT
#CFLAGS += -DUSE_IRC

LDFLAGS := $(CPU_FLAGS)
LDFLAGS += -mthumb -mcpu=cortex-m0plus -mlittle-endian
LDFLAGS += -Wl,-T,$(LINKER_SCRIPT) -Wl,-nostdlib -Wl,--warn-common
LDFLAGS += -Wl,--gc-sections -Wl,-O3
LDFLAGS += -Wl,-Map=$(TARGET_MAP) -Wl,--cref

LDLIBS := $(addprefix -l,$(LIBS))


###############################################################################
# Plumbing for rules
vpath %.c $(SOURCE_DIRS)

dummy := $(shell $(MKDIR_P) $(BUILD_DIR))   # Always create the build directory

define compile-objects
$1/%.o: %.c
	$(ECHO) [CC] $$<
	$(QUIET) $(CC) $(CFLAGS) -c $$< -o $$@

$1/%.lst: %.c
	$(ECHO) [LIST] $$<
	$(QUIET) $(CC) -c -g -Wa,-adlhn $(CFLAGS) $$< -o /dev/null > $$@
endef

$(foreach bdir, $(BUILD_DIR), $(eval $(call compile-objects,$(bdir))))


###############################################################################
# Rules
all : $(TARGET_BIN) $(TARGET_HEX)

$(TARGET_ELF): $(OBJECTS) $(LINKER_SCRIPT)
	$(ECHO) [LD] $@
	$(QUIET) $(LD) $(LDFLAGS) -o $@ $(OBJECTS) $(LDLIBS)

$(TARGET_BIN): $(TARGET_ELF)
	$(ECHO) [CP] $@
	$(QUIET) $(OBJCOPY) $< -O binary $@

$(TARGET_HEX): $(TARGET_ELF)
	$(ECHO) [CP] $@
	$(QUIET) $(OBJCOPY) --remove-section=.persistent_data $< -O ihex $@
##--remove-section=.persistent_data

# Create list files that include C code as well as Assembler
list: $(OBJECTS:.o=.lst)

# Print a memory usage summary
summary: $(TARGET_MAP)
	$(QUIET) $(MAP_SUMMARY_TOOL) $<

# Invoke the tool to program the microcontroller
program: $(TARGET_HEX)
	$(QUIET )$(FLASH_TOOL) $<

# Invoke a tool for UART communication
terminal:
	$(QUIET) $(TERMINAL_PROGRAM)

# Clean all generated files
clean:
	$(ECHO) [RM] $(BUILD_DIR)
	$(QUIET) $(RM) -rf $(BUILD_DIR)/*


.PHONY : all clean program terminal list summary
