#
# Copyright 2007-2011 Clozure Associates
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This makefile is written to be used with the mingw toolchain
# running in the Cygwin environment or Msys2 environment.

# If running in the Cygwin environment:
# Install the following Cygwin packages:
# * make
# * m4
# * mingw64-x86_64-binutils
# * mingw64-x86_64-gcc-core
# * mingw64-x86_64-headers
# * mingw64-x86_64-pthreads
# * mingw64-x86_64-runtime
#
# Accept the dependencies the installation tool may add.

# If running in the Msys2 environment:
# Install the following Msys2 packages using pacman -S:
# * make
# * m4
# * mingw-w64-x86_64-binutils
# * mingw-w64-x86_64-crt
# * mingw-w64-x86_64-gcc
# * mingw-w64-x86_64-headers
# * mingw-w64-x86_64-winpthreads
# Accept the dependencies the installation tool may add.
# Currently only install make, m4 and mingw-w64-x86_64-gcc will install other
# mingw-w64-* dependencies. May change in the future.
# Running make using msys2-installation-directory/mingw64.exe, not msys2.exe.
VC_REVISION := "$(shell git describe --dirty 2>/dev/null || echo unknown)"

VPATH = ../
RM = rm
# gcc64, as64: until there's a real win64 gcc, assume that gcc and gas
# are installed under these names

ifeq ($(MSYSTEM),)
CC = x86_64-w64-mingw32-gcc
AS = x86_64-w64-mingw32-as
LD = x86_64-w64-mingw32-ld
else
CC = gcc
AS = as
LD = ld
endif

M4 = m4

ASFLAGS = -g --64
M4FLAGS = -DWIN_64 -DWINDOWS -DX86 -DX8664 -DHAVE_TLS -DEMUTLS -DTCR_IN_GPR
CDEFINES = -DWIN_64 -DWINDOWS -D_REENTRANT -DX86 -DX8664 -D_GNU_SOURCE -DHAVE_TLS -DEMUTLS -DTCR_IN_GPR -DVC_REVISION=$(VC_REVISION)
CDEBUG = -g
COPT = -O2
# Once in a while, -Wformat says something useful.  The odds are against that,
# however.
WFORMAT = -Wno-format
PLATFORM_H = platform-win64.h

# If the linker supports a "--hash-style=" option, use traditional
# SysV hash tables.  (If it doesn't support that option, assume
# that traditional hash tables will be used by default.)
ld_has_hash_style = $(shell $(LD) --help | grep "hash-style=")
ifeq ($(ld_has_hash_style),)
HASH_STYLE=
else
HASH_STYLE="-Wl,--hash-style=sysv"
endif

# There may be some confusion about whether or not C symbols have
# leading underscores or not.  The assembler sources seem to
# expect them to and mingw import libs seem to use them, but
# it's not clear whether or not native win64 libraries use this
# convention (and I'm not sure whether the Cygwin-hosted win64
# toolchain behaves the same was as when hosted on Linux ...
# The compiler default seems to be to use them; if we want to
# suppress their use, uncomment the following:

SUPPRESS_UNDERSCORES=#-fno-leading-underscore

.s.o:
	$(M4) $(M4FLAGS) -I../ $< | $(AS)  $(ASFLAGS) -o $@
.c.o:
	$(CC) -include ../$(PLATFORM_H) -c $< $(CDEFINES) $(CDEBUG) $(COPT) $(WFORMAT) ${SUPPRESS_UNDERSCORES} -m64 -o $@

SPOBJ = pad.o x86-spjump64.o x86-spentry64.o x86-subprims64.o
ASMOBJ = x86-asmutils64.o imports.o

COBJ  = pmcl-kernel.o gc-common.o x86-gc.o bits.o  x86-exceptions.o \
	x86-utils.o \
	image.o thread_manager.o lisp-debug.o memory.o windows-calls.o

DEBUGOBJ = lispdcmd.o plprint.o plsym.o xlbt.o x86_print.o
KERNELOBJ= $(COBJ) x86-asmutils64.o  imports.o

SPINC =	lisp.s m4macros.m4 x86-constants.s x86-macros.s errors.s x86-uuo.s \
	x86-constants64.s lisp_globals.s

CHEADERS = area.h bits.h x86-constants.h lisp-errors.h gc.h lisp.h \
	lisp-exceptions.h lisp_globals.h macros.h memprotect.h image.h \
	threads.h x86-constants64.h x86-exceptions.h lisptypes.h \
	x86-utils.h \
	$(PLATFORM_H) constants.h os-windows.h


KSPOBJ = $(SPOBJ)
all:	../../wx86cl64.exe


OSLIBS = -lpsapi -lws2_32 -static -lpthread


../../wx86cl64.exe:	$(KSPOBJ) $(KERNELOBJ) $(DEBUGOBJ) Makefile pei-x86-64.x
	$(CC) -Wl,--image-base=0x10000 -Wl,-script=pei-x86-64.x -m64 $(CDEBUG)   $(HASH_STYLE) -o $@ $(USE_LINK_MAP) $(KSPOBJ) $(KERNELOBJ) $(DEBUGOBJ) $(OSLIBS)


$(SPOBJ): $(SPINC)
$(ASMOBJ): $(SPINC)
$(COBJ): $(CHEADERS)
$(DEBUGOBJ): $(CHEADERS) lispdcmd.h


cclean:
	$(RM) -f $(KERNELOBJ) $(DEBUGOBJ) ../../wx86cl64.exe

clean:	cclean
	$(RM) -f $(SPOBJ)

strip:	../../wx86cl64.exe
	strip -g ../../wx86cl64.exe
