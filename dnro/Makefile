TOP := $(CURDIR)

ifeq ($(filter clean,$(MAKECMDGOALS)),)
ifndef FFPREFIX
$(error FFPREFIX is not set - pass FFPREFIX=<ffmpeg install prefix>)
endif
ifndef DLINK
$(error DLINK is not set - pass DLINK=<libnx-dlink checkout>)
endif
endif

DEVKITPRO ?= /opt/devkitpro
DKA64 := $(DEVKITPRO)/devkitA64
CC := $(DKA64)/bin/aarch64-none-elf-gcc
PY := python3

ARCH := -march=armv8-a -mtune=cortex-a57 -mtp=soft -fPIC
BASEFLAGS := -g -O2 -ffunction-sections -fdata-sections $(ARCH) -D__SWITCH__ -I$(DEVKITPRO)/libnx/include
CFLAGS := $(BASEFLAGS)
ASFLAGS := -g $(ARCH)
LDFLAGS := -specs=$(DLINK)/tools/module.specs -g $(ARCH) -shared -nodefaultlibs -Wl,--export-dynamic -Wl,-z,nopack-relative-relocs -Wl,-Map,$(TOP)/ffmpeg.map -Wl,--gc-sections -Wl,-Bsymbolic

FFLIBS := $(FFPREFIX)/lib/libavformat.a $(FFPREFIX)/lib/libavcodec.a $(FFPREFIX)/lib/libswscale.a $(FFPREFIX)/lib/libswresample.a $(FFPREFIX)/lib/libavutil.a

all: $(TOP)/ffmpeg.dnro $(TOP)/ffmpeg.undef $(TOP)/ffmpeg.def

$(TOP)/ffmpeg.def: $(TOP)/ffmpeg.elf
	$(DKA64)/bin/aarch64-none-elf-nm -D --defined-only $< | awk '{print $$NF}' | sort -u > $@

$(TOP)/ffmpeg.undef: $(TOP)/ffmpeg.elf
	$(DKA64)/bin/aarch64-none-elf-nm -u -D $< | awk '{print $$NF}' | sort -u > $@

$(TOP)/ffmpeg.dnro: $(TOP)/ffmpeg.elf
	$(PY) $(DLINK)/tools/elf2dnro.py $(TOP)/ffmpeg.elf $(TOP)/ffmpeg.dnro

$(TOP)/ffmpeg.elf: $(TOP)/build/stub/stub.o $(TOP)/build/stub/crt0.o $(FFLIBS)
	$(CC) $(LDFLAGS) -o $@ $(TOP)/build/stub/stub.o $(TOP)/build/stub/crt0.o \
		-Wl,--whole-archive $(FFLIBS) -Wl,--no-whole-archive -lgcc

$(TOP)/build/stub/stub.o: $(TOP)/stub/stub.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $< -o $@

$(TOP)/build/stub/crt0.o: $(TOP)/stub/crt0.s
	@mkdir -p $(dir $@)
	$(CC) $(ASFLAGS) -c $< -o $@

clean:
	rm -rf $(TOP)/build $(TOP)/ffmpeg.elf $(TOP)/ffmpeg.dnro $(TOP)/ffmpeg.map $(TOP)/ffmpeg.undef $(TOP)/ffmpeg.def

.PHONY: all clean
