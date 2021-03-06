CROSS_COMPILE ?= arm-none-eabi-
CC = $(CROSS_COMPILE)gcc
OBJCOPY = $(CROSS_COMPILE)objcopy
TOOLCHAIN_DIR=$(shell dirname `which arm-none-eabi-gcc`)
ifeq ($(TOOLCHAIN_DIR),)
TOOLCHAIN_DIR=$(shell find -L /opt/paparazzi/arm-multilib ~/sat /opt/paparazzi/stm32 -maxdepth 1 -type d -name arm-none-eabi 2>/dev/null | head -n 1)
CC = $(TOOLCHAIN_DIR)/../bin/$(CROSS_COMPILE)gcc
OBJCOPY = $(TOOLCHAIN_DIR)/../bin/$(CROSS_COMPILE)objcopy
endif

LIBOPENCM3 ?= $(TOOLCHAIN_DIR)/..
OOCD_INTERFACE  ?= flossjtag
OOCD_TARGET     ?= stm32
OOCD = openocd
GDB = $(TOOLCHAIN_DIR)/arm-none-eabi-gdb
BMP_PORT =
VERSION = V1.0
DEV_SERIAL = NSERIAL

CFLAGS += -Os -g3 -Istm32/include -mcpu=cortex-m3 -mthumb -msoft-float -DSTM32F1 -I$(LIBOPENCM3)/include --function-sections --data-sections -DVERSION="\"$(VERSION)\"" -DDEV_SERIAL="\"$(DEV_SERIAL)\""
LDFLAGS_BOOT = -lopencm3_stm32 -Wl,--defsym,_stack=0x20005000 \
	-Wl,-T,luftboot.ld -nostartfiles -lc -lnosys -Wl,-Map=mapfile \
	-mthumb -march=armv7 -mcpu=cortex-m3 -mfix-cortex-m3-ldrd -msoft-float -L$(LIBOPENCM3)/lib/stm32 -Wl,--gc-sections
LDFLAGS_BOOT += -Wl,--section-start=.devserial=0x8001FF0
LDFLAGS = $(LDFLAGS_BOOT) -Wl,-Ttext=0x8002000

ifneq ($(V),1)
Q := @
NULL := 2>/dev/null
else
LDFLAGS_BOOT += -Wl,--print-gc-sections
endif

all: luftboot.bin luftboot.hex

luftboot: luftboot.o
	$(CC) $^ -o $@ $(LDFLAGS_BOOT)

luftboot.bin: luftboot
	$(OBJCOPY) -O binary $^ $@

luftboot.hex: luftboot
	$(OBJCOPY) -O ihex $^ $@

clean:
	-rm -rf luftboot.o luftboot luftboot.bin luftboot.hex mapfile

ifeq ($(BMP_PORT),)
flash: luftboot.hex
	@echo "  OOCD  luftboot.hex"
	$(Q)$(OOCD) -f interface/$(OOCD_INTERFACE).cfg \
		-f target/$(OOCD_TARGET).cfg \
		-c init \
		-c "reset init" \
		-c "stm32f1x mass_erase 0" \
		-c "flash write_image luftboot.hex" \
		-c reset \
		-c shutdown
else
flash: luftboot
	@echo "Using Black Magic Probe"
	@echo "Using GDB = $(GDB)"
	@echo "  BMP\t$<"
	$(Q)$(GDB) --batch \
		-ex 'target extended-remote $(BMP_PORT)' \
		-x /Users/esdentem/sat/arm-none-eabi/share/libopencm3/scripts/black_magic_probe_flash.scr \
		$<
endif
