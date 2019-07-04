# Toolchain
CC:=avr-gcc
CPP:=avr-cpp
AS:=avr-as
LD:=avr-ld
OBJCOPY:=avr-objcopy
OBJDUMP:=avr-objdump
MICRONUCLEUS:=micronucleus

# Command line flags

DEBUG?=0
MCU?=attiny85

.PHONY: help
help:
	$(info Usage: make [VARIABLE=VALUE] TARGET)
	$(info Variables:)
	$(info - DEBUG=1: Enable debugging output (.map/.lst) and logging)
	$(info - MCU=[MCU]: Set target MCU (only attiny85 is supported at the moment))
	$(info Targets:)
	$(info - all: Build V-USB and GCSpark, link and create .bin and .hex)
	$(info - clean: Remove all build files)
	$(info - vusb: Only build V-USB object files)
	$(info - gcspark: Only build GCSpark object files)
	$(info - size: Show memory usage)
	$(info - flash: Flash .hex using micronucleus)

# V-USB paths

VUSB_SRCDIR?=external/v-usb/usbdrv
VUSB_OBJECTS:=$(addprefix $(VUSB_SRCDIR)/,\
usbdrv.o \
usbdrvasm.o \
)

# GCSpark paths

GCSPARK_SRCDIR?=src
GCSPARK_BINDIR?=bin
GCSPARK_OBJECTS:=$(addprefix $(GCSPARK_SRCDIR)/,\
main.o \
)

GCSPARK_OUTPUT?=$(GCSPARK_BINDIR)/gcspark
GCSPARK_OUTPUTS=$(addprefix $(GCSPARK_OUTPUT), .elf .hex .bin)
ifeq ($(DEBUG), 1)
GCSPARK_OUTPUTS+=$(addprefix $(GCSPARK_OUTPUT), .lst .map)
endif

# Tool flags

INCLUDES:=-I$(VUSB_SRCDIR) -I$(GCSPARK_SRCDIR)
DEFINES:=-DF_CPU=16500000
# C compiler: All warnings, optimize for size, force function and data sections
CFLAGS:=-mmcu=$(MCU) -Wall -Os -ffunction-sections -fdata-sections
ifeq ($(DEBUG), 1)
CFLAGS+=-g -gstabs
endif
# C preprocessor
CPPFLAGS:=-mmcu=$(MCU) $(INCLUDES) $(DEFINES)
# Assembler
ASFLAGS:=-mmcu=$(MCU) --warn
ifeq ($(DEBUG), 1)
ASFLAGS+=-g
endif
# Linker: Remove unused sections
LDFLAGS:=-gc-sections

# Preprocess .S files in separate step and then assemble instead of using $(CC)

.SUFFIXES:
.SUFFIXES: .c .o .s

%.s: %.S
	$(PREPROCESS.S) $< > $@

# Implicit rules for outputs derived from .elf
%.hex: %.elf
	$(OBJCOPY) -j .text -j .data -O ihex $< $@

%.bin: %.elf
	$(OBJCOPY) -j .text -j .data -O binary $< $@

%.lst: %.elf
	$(OBJDUMP) -h -S $< > $@

# Explicit rule for .elf
ifeq ($(DEBUG), 1)
$(GCSPARK_OUTPUT).elf $(GCSPARK_OUTPUT).map: $(VUSB_OBJECTS) $(GCSPARK_OBJECTS)
	$(LD) $(LDFLAGS) -Map $(GCSPARK_OUTPUT).map -o $@ $^
else
$(GCSPARK_OUTPUT).elf: $(VUSB_OBJECTS) $(GCSPARK_OBJECTS)
	$(LD) $(LDFLAGS) -o $@ $^
endif

.PHONY: vusb gcspark all flash size clean
vusb: $(VUSB_OBJECTS)
gcspark: $(GCSPARK_OBJECTS)
all: vusb gcspark $(GCSPARK_OUTPUTS)

flash: $(GCSPARK_OUTPUT).hex
	$(MICRONUCLEUS) --type hex --run $^

size: $(GCSPARK_OUTPUT).elf
	avr-size -C --mcu=$(MCU) $^

clean:
	$(RM) $(VUSB_OBJECTS)
	$(RM) $(GCSPARK_OBJECTS)
	$(RM) $(GCSPARK_OUTPUTS)
