# Software Name : bmc-cache
# SPDX-FileCopyrightText: Copyright (c) 2021 Orange
# SPDX-License-Identifier: LGPL-2.1-only
#
# This software is distributed under the
# GNU Lesser General Public License v2.1 only.
#
# Author: Yoann GHIGOFF <yoann.ghigoff@orange.com> et al.
#
#	To use this Makefile: clang and llvm must be installed,
#	kernel sources available under ./linux and libbpf statically
#	compiled in Linux source tree.
#
#	bmc_kern.c depends on kernel headers and bpf_helpers.h
#	bmc_user.c depends on libbpf

LINUX_SRC ?= ./linux
LINUX_OBJ = $(LINUX_SRC)/../build/linux
LINUX_TOOLS_PATH = $(LINUX_SRC)/tools
LINUX_LIB_PATH = $(LINUX_TOOLS_PATH)/lib
LIBBPF_PATH = $(LINUX_LIB_PATH)/bpf

TARGETS += fast

CLANG ?= clang
CC := gcc

KERN_SOURCES = ${TARGETS:=_kern.c}
USER_SOURCES = ${TARGETS:=_user.c}
KERN_OBJECTS = ${KERN_SOURCES:.c=.o}
USER_OBJECTS = ${USER_SOURCES:.c=.o}

override CFLAGS += -g -O3 -Wall -DKBUILD_MODNAME="\"wzz\""
override CFLAGS += -I$(LINUX_LIB_PATH)
LDFLAGS ?= -L$(LINUX_OBJ) -Wl,-rpath=$(LINUX_OBJ) -lbpf -lelf -lz

LINUXINCLUDE := -I$(LINUX_OBJ)/usr/include

override EXTRA_CFLAGS += -Werror

BPF_CFLAGS = -fno-stack-protector -O3 -g -target bpf

###

all: $(TARGETS) $(KERN_OBJECTS)

.PHONY: clean

clean:
	@find . -type f \
		\( -name '*~' \
		-o -name '*.ll' \
		-o -name '*.bc' \
		-o -name 'core' \) \
		-exec rm -vf '{}' \;
	rm -f $(TARGETS)
	rm -f $(KERN_OBJECTS)
	rm -f $(USER_OBJECTS)
	rm -f $(OBJECT_LOADBPF)

$(LIBBPF): $(wildcard $(LIBBPF_PATH)/*.[ch] $(LIBBPF_PATH)/Makefile)
	make -C $(LIBBPF_PATH)

# Compiling of eBPF restricted-C code with LLVM
#  clang option -S generated output file with suffix .ll
#   which is the non-binary LLVM assembly language format
#   (normally LLVM bitcode format .bc is generated)
#
# Use -Wno-address-of-packed-member as eBPF verifier enforces
# unaligned access checks where necessary
#
$(KERN_OBJECTS): %.o: %.c
	$(CLANG) $(BPF_CFLAGS) $(LINUXINCLUDE) $(EXTRA_CFLAGS) -c $< -o $@

$(TARGETS): %: %_user.c $(OBJECTS)
	$(CC) $(CFLAGS) $(OBJECTS) -o $@ $< $(LDFLAGS)
