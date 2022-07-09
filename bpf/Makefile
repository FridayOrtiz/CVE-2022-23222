ARCH ?= x86_64
CC = clang-8
LLC = llc-8
OPT = opt-8
LLVM_DIS = llvm-dis-8
LLVM_OBJCOPY = llvm-objcopy-8

SOURCES ?= filter.c

CFLAGS += \
	-D__KERNEL__ \
	-D__BPF_TRACING__ \
	-Wunused \
	-Wall \
	-Werror \
	-Wno-pointer-sign \
	-Wno-address-of-packed-member \
	-Wno-compare-distinct-pointer-types \
	-Wno-gnu-variable-sized-type-not-at-end \
	-Wno-sometimes-uninitialized \
	-Wno-tautological-compare \
	-fno-stack-protector \
	-Xclang -disable-llvm-passes \
	-O2

CFLAGS += -D__ASM_SYSREG_H
KERNEL_ARCH_NAME = x86
KERNEL_HEADER_VERSION ?= 5.4.0-58-generic
TARGET = -target x86_64

CFLAGS += -D__TARGET_ARCH_$(KERNEL_ARCH_NAME)

KERNEL_HEADERS_ROOT ?= /usr/src/linux-headers-$(KERNEL_HEADER_VERSION)

INCLUDES = -I src/ \
	-I $(KERNEL_HEADERS_ROOT)/arch/$(KERNEL_ARCH_NAME)/include \
	-I $(KERNEL_HEADERS_ROOT)/arch/$(KERNEL_ARCH_NAME)/include/uapi \
	-I $(KERNEL_HEADERS_ROOT)/arch/$(KERNEL_ARCH_NAME)/include/generated \
	-I $(KERNEL_HEADERS_ROOT)/arch/$(KERNEL_ARCH_NAME)/include/generated/uapi \
	-I $(KERNEL_HEADERS_ROOT)/include \
	-I $(KERNEL_HEADERS_ROOT)/include/uapi \
	-I $(KERNEL_HEADERS_ROOT)/include/generated \
	-I $(KERNEL_HEADERS_ROOT)/include/generated/uapi

all:
#	$(CC) $(TARGET) $(CFLAGS) -c $(SOURCES) $(INCLUDES) -o filter_program_$(ARCH)
	$(CC) $(TARGET) $(CFLAGS) -emit-llvm -c $(SOURCES) $(INCLUDES) -o - | \
		$(OPT) -O2 -mtriple=bpf-pc-linux | $(LLVM_DIS) | \
		$(LLC) -march=bpf -filetype=obj -o - | \
		$(LLVM_OBJCOPY) -R ".BTF" -R ".BTF.ext" - filter_program_$(ARCH)

clean:
	rm -rf filter_program_*

depends:
	apt-get update
	apt-get install -y llvm-8 clang-8 libclang-8-dev \
		linux-headers-5.4.0-58-generic \
		make binutils curl coreutils

.PHONY: all clean depends