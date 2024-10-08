#/* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
# *   Mupen64plus-video-glide64mk2 - Makefile                               *
# *   Mupen64Plus homepage: https://mupen64plus.org/                        *
# *   Copyright (C) 2010 Jon Ring                                           *
# *   Copyright (C) 2007-2009 Richard Goedeken                              *
# *   Copyright (C) 2007-2008 DarkJeztr Tillin9                             *
# *                                                                         *
# *   This program is free software; you can redistribute it and/or modify  *
# *   it under the terms of the GNU General Public License as published by  *
# *   the Free Software Foundation; either version 2 of the License, or     *
# *   (at your option) any later version.                                   *
# *                                                                         *
# *   This program is distributed in the hope that it will be useful,       *
# *   but WITHOUT ANY WARRANTY; without even the implied warranty of        *
# *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         *
# *   GNU General Public License for more details.                          *
# *                                                                         *
# *   You should have received a copy of the GNU General Public License     *
# *   along with this program; if not, write to the                         *
# *   Free Software Foundation, Inc.,                                       *
# *   51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.          *
# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * */
# Makefile for Glide64mk2 plugin in Mupen64Plus

# detect operating system
UNAME ?= $(shell uname -s)
OS := NONE
ifeq ("$(UNAME)","Linux")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(UNAME)","linux")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifneq ("$(filter GNU hurd,$(UNAME))","")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(UNAME)","Darwin")
  OS = OSX
  SO_EXTENSION = dylib
  SHARED = -bundle
endif
ifeq ("$(UNAME)","FreeBSD")
  OS = FREEBSD
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(UNAME)","OpenBSD")
  OS = FREEBSD
  SO_EXTENSION = so
  SHARED = -shared
endif
ifneq ("$(filter GNU/kFreeBSD kfreebsd,$(UNAME))","")
  OS = LINUX
  SO_EXTENSION = so
  SHARED = -shared
endif
ifeq ("$(patsubst MINGW%,MINGW,$(UNAME))","MINGW")
  OS = MINGW
  SO_EXTENSION = dll
  SHARED = -shared
  PIC = 0
  CPPFLAGS += -DNO_FILTER_THREAD
endif
ifeq ("$(OS)","NONE")
  $(error OS type "$(UNAME)" not supported.  Please file bug report at 'https://github.com/mupen64plus/mupen64plus-core/issues')
endif

# detect system architecture
HOST_CPU ?= $(shell uname -m)
CPU := NONE
ifneq ("$(filter x86_64 amd64,$(HOST_CPU))","")
  CPU := X86
  ifeq ("$(BITS)", "32")
    ARCH_DETECTED := 64BITS_32
    PIC ?= 0
  else
    ARCH_DETECTED := 64BITS
    PIC ?= 1
  endif
endif
ifneq ("$(filter pentium i%86,$(HOST_CPU))","")
  CPU := X86
  ARCH_DETECTED := 32BITS
  PIC ?= 0
endif
ifneq ("$(filter ppc macppc socppc powerpc,$(HOST_CPU))","")
  CPU := PPC
  ARCH_DETECTED := 32BITS
  BIG_ENDIAN := 1
  PIC ?= 1
  NO_ASM := 1
  NO_SSE := 1
  $(warning Architecture "$(HOST_CPU)" not officially supported.')
endif
ifneq ("$(filter ppc64 powerpc64,$(HOST_CPU))","")
  CPU := PPC
  ARCH_DETECTED := 64BITS
  BIG_ENDIAN := 1
  PIC ?= 1
  NO_ASM := 1
  $(warning Architecture "$(HOST_CPU)" not officially supported.')
endif
ifneq ("$(filter ppc64le powerpc64le,$(HOST_CPU))","")
  CPU := PPC
  ARCH_DETECTED := 64BITS
  BIG_ENDIAN := 0
  PIC ?= 1
  NO_ASM := 1
  NO_SSE := 1
  $(warning Architecture "$(HOST_CPU)" not officially supported.')
endif
ifneq ("$(filter arm%,$(HOST_CPU))","")
  ifeq ("$(filter arm%b,$(HOST_CPU))","")
    CPU := ARM
    ARCH_DETECTED := 32BITS
    PIC ?= 1
    NO_ASM := 1
    NO_SSE := 1
    $(warning Architecture "$(HOST_CPU)" not officially supported.')
  endif
endif
ifneq ("$(filter aarch64,$(HOST_CPU))","")
    CPU := AARCH
    ARCH_DETECTED := 64BITS
    PIC ?= 1
    NEW_DYNAREC := 1
    NO_ASM := 1
    NO_SSE := 1
endif
ifneq ("$(filter riscv64,$(HOST_CPU))","")
    CPU := RISCV64
    ARCH_DETECTED := 64BITS
    PIC ?= 1
    NO_ASM := 1
    NO_SSE := 1
    $(warning Architecture "$(HOST_CPU)" not officially supported.)
endif
ifeq ("$(CPU)","NONE")
  $(error CPU type "$(HOST_CPU)" not supported.  Please file bug report at 'https://github.com/mupen64plus/mupen64plus-core/issues')
endif

SRCDIR = ../../src
OBJDIR = _obj$(POSTFIX)

# base CFLAGS, LDLIBS, and LDFLAGS
OPTFLAGS ?= -O3 -flto
WARNFLAGS ?= -Wall -Wno-unused-function
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) -ffast-math -fno-strict-aliasing -fvisibility=hidden -I$(SRCDIR) -I$(SRCDIR)/Glitch64/inc -DGCC
CXXFLAGS += -fvisibility-inlines-hidden -std=gnu++17
LDFLAGS += $(SHARED)

ifeq ($(CPU), X86)
  CFLAGS +=  -mmmx -msse
endif

# Since we are building a shared library, we must compile with -fPIC on some architectures
# On 32-bit x86 systems we do not want to use -fPIC because we don't have to and it has a big performance penalty on this arch
ifeq ($(PIC), 1)
  CFLAGS += -fPIC
else
  CFLAGS += -fno-PIC
endif

ifeq ($(BIG_ENDIAN), 1)
  CFLAGS += -DM64P_BIG_ENDIAN
endif

# tweak flags for 32-bit build on 64-bit system
ifeq ($(ARCH_DETECTED), 64BITS_32)
  ifeq ($(OS), FREEBSD)
    $(error Do not use the BITS=32 option with FreeBSD, use -m32 and -m elf_i386)
  endif
  ifneq ($(OS), OSX)
    ifeq ($(OS), MINGW)
      LDFLAGS += -Wl,-m,i386pe
    else
      CFLAGS += -m32
      LDFLAGS += -Wl,-m,elf_i386
    endif
  endif
endif

ifeq ($(ARCH_DETECTED), 64BITS)
  ifeq ($(OS), MINGW)
    LDFLAGS += -Wl,-m,i386pep
  endif
endif

ifeq ($(origin PKG_CONFIG), undefined)
  PKG_CONFIG = $(CROSS_COMPILE)pkg-config
  ifeq ($(shell which $(PKG_CONFIG) 2>/dev/null),)
    $(error $(PKG_CONFIG) not found)
  endif
endif

# set special flags per-system
ifeq ($(OS), LINUX)
  # only export api symbols
  LDFLAGS += -Wl,-version-script,$(SRCDIR)/video_api_export.ver
  LDLIBS += -ldl
endif
ifeq ($(OS), OSX)
  OSX_SDK_PATH = $(shell xcrun --sdk macosx --show-sdk-path)
  ifeq ($(CPU), X86)
    LDFLAGS += -bundle -L/opt/local/lib
    CXXFLAGS += -stdlib=libc++
    LDLIBS += -ldl
    ifeq ($(ARCH_DETECTED), 64BITS)
      CFLAGS += -pipe -I /opt/local/include -arch x86_64 -mmacosx-version-min=10.13 -isysroot $(OSX_SDK_PATH)
    else
      CFLAGS += -pipe -I /opt/local/include -mmmx -msse -fomit-frame-pointer -arch i686 -mmacosx-version-min=10.13 -isysroot $(OSX_SDK_PATH)
    endif
  endif
  # use system zlib on OSX
  ZLIB_LDLIBS += -lz
endif
ifeq ($(OS), FREEBSD)
  LDLIBS += -lc
endif
ifeq ($(OS), MINGW)
  ifeq ($(origin GLEW_CFLAGS) $(origin GLEW_LDLIBS), undefined undefined)
    ifeq ($(shell $(PKG_CONFIG) --modversion glew 2>/dev/null),)
      $(error No GLEW development libraries found!)
    endif
    GLEW_CFLAGS += $(shell $(PKG_CONFIG) --cflags glew)
    GLEW_LDLIBS +=  $(shell $(PKG_CONFIG) --libs glew)
  endif
  CFLAGS += $(GLEW_CFLAGS)
  LDLIBS += $(GLEW_LDLIBS)
endif

ifeq ($(origin ZLIB_CFLAGS) $(origin ZLIB_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion zlib 2>/dev/null),)
    $(error No zlib development libraries found!)
  endif
  ZLIB_CFLAGS += $(shell $(PKG_CONFIG) --cflags zlib)
  ZLIB_LDLIBS +=  $(shell $(PKG_CONFIG) --libs zlib)
endif
CFLAGS += $(ZLIB_CFLAGS)
LDLIBS += $(ZLIB_LDLIBS)

ifeq ($(origin LIBPNG_CFLAGS) $(origin LIBPNG_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion libpng 2>/dev/null),)
    $(error No libpng development libraries found!)
  endif
  LIBPNG_CFLAGS += $(shell $(PKG_CONFIG) --cflags libpng)
  LIBPNG_LDLIBS +=  $(shell $(PKG_CONFIG) --libs libpng)
endif
CFLAGS += $(LIBPNG_CFLAGS)
LDLIBS += $(LIBPNG_LDLIBS)

# search for OpenGL libraries
ifeq ($(VC), 1)
  GL_CFLAGS += -I/opt/vc/include -I/opt/vc/include/interface/vcos/pthreads -I/opt/vc/include/interface/vmcs_host/linux
  GL_LDLIBS += -lm -L/opt/vc/lib -lEGL -lbcm_host
  USE_GLES=1
endif
ifeq ($(USE_GLES), 1)
  CFLAGS += -DUSE_GLES
  GL_LDLIBS += -lGLESv2
endif
ifeq ($(OS), OSX)
  GL_LDLIBS = -framework OpenGL
endif
ifeq ($(OS), MINGW)
  GL_LDLIBS = -lopengl32
endif
ifeq ($(origin GL_CFLAGS) $(origin GL_LDLIBS), undefined undefined)
  ifeq ($(shell $(PKG_CONFIG) --modversion gl 2>/dev/null),)
    $(error No OpenGL development libraries found!)
  endif
  GL_CFLAGS += $(shell $(PKG_CONFIG) --cflags gl)
  GL_LDLIBS +=  $(shell $(PKG_CONFIG) --libs gl)
endif
CFLAGS += $(GL_CFLAGS)
LDLIBS += $(GL_LDLIBS)

# test for presence of SDL
ifeq ($(origin SDL_CFLAGS) $(origin SDL_LDLIBS), undefined undefined)
  SDL_CONFIG = $(CROSS_COMPILE)sdl2-config
  ifeq ($(shell which $(SDL_CONFIG) 2>/dev/null),)
    SDL_CONFIG = $(CROSS_COMPILE)sdl-config
    ifeq ($(shell which $(SDL_CONFIG) 2>/dev/null),)
      $(error No SDL development libraries found!)
    else
      $(warning Using SDL 1.2 libraries)
    endif
  endif
  SDL_CFLAGS  += $(shell $(SDL_CONFIG) --cflags)
  SDL_LDLIBS += $(shell $(SDL_CONFIG) --libs)
  ifeq ($(OS),OSX)
    SDL_LDLIBS += -Wl,-framework,Cocoa
  endif
endif
CFLAGS += $(SDL_CFLAGS)
LDLIBS += $(SDL_LDLIBS)

ifneq ($(OS), MINGW)
  CPPFLAGS += -D__unix__
endif

# set mupen64plus core API header path
ifneq ("$(APIDIR)","")
  CFLAGS += "-I$(APIDIR)"
else
  TRYDIR = ../../../mupen64plus-core/src/api
  ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
    CFLAGS += -I$(TRYDIR)
  else
    TRYDIR = /usr/local/include/mupen64plus
    ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
      CFLAGS += -I$(TRYDIR)
    else
      TRYDIR = /usr/include/mupen64plus
      ifneq ("$(wildcard $(TRYDIR)/m64p_types.h)","")
        CFLAGS += -I$(TRYDIR)
      else
        $(error Mupen64Plus API header files not found! Use makefile parameter APIDIR to force a location.)
      endif
    endif
  endif
endif

# reduced compile output when running make without V=1
ifneq ($(findstring $(MAKEFLAGS),s),s)
ifndef V
	Q_CC  = @echo '    CC  '$@;
	Q_CXX = @echo '    CXX '$@;
	Q_LD  = @echo '    LD  '$@;
endif
endif

# set base program pointers and flags
CC        = $(CROSS_COMPILE)gcc
CXX       = $(CROSS_COMPILE)g++
RM       ?= rm -f
INSTALL  ?= install
MKDIR ?= mkdir -p
COMPILE.c = $(Q_CC)$(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
COMPILE.cc = $(Q_CXX)$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
LINK.o = $(Q_LD)$(CXX) $(CXXFLAGS) $(LDFLAGS) $(TARGET_ARCH)

# set special flags for given Makefile parameters
ifeq ($(DEBUG),1)
  CFLAGS += -g
  INSTALL_STRIP_FLAG ?= 
else
  ifneq ($(OS),OSX)
    INSTALL_STRIP_FLAG ?= -s
  endif
endif
ifeq ($(NO_ASM), 1)
  CFLAGS += -DNO_ASM
endif
ifeq ($(NO_SSE), 1)
  CFLAGS += -DNOSSE
endif
ifeq ($(USE_FRAMESKIPPER), 1)
  CFLAGS += -DUSE_FRAMESKIPPER
endif

# set installation options
ifeq ($(PREFIX),)
  PREFIX := /usr/local
endif
ifeq ($(SHAREDIR),)
  SHAREDIR := $(PREFIX)/share/mupen64plus
endif
ifeq ($(LIBDIR),)
  LIBDIR := $(PREFIX)/lib
endif
ifeq ($(PLUGINDIR),)
  PLUGINDIR := $(LIBDIR)/mupen64plus
endif

# list of source files to compile
SOURCE = \
	$(SRCDIR)/Glide64/3dmath.cpp \
	$(SRCDIR)/Glide64/Combine.cpp \
	$(SRCDIR)/Glide64/Config.cpp \
	$(SRCDIR)/Glide64/CRC.cpp \
	$(SRCDIR)/Glide64/Debugger.cpp \
	$(SRCDIR)/Glide64/DepthBufferRender.cpp \
	$(SRCDIR)/Glide64/FBtoScreen.cpp \
	$(SRCDIR)/Glide64/Ini.cpp \
	$(SRCDIR)/Glide64/Keys.cpp \
	$(SRCDIR)/Glide64/Main.cpp \
	$(SRCDIR)/Glide64/rdp.cpp \
	$(SRCDIR)/Glide64/TexBuffer.cpp \
	$(SRCDIR)/Glide64/TexCache.cpp \
	$(SRCDIR)/Glide64/Util.cpp

ifeq ($(USE_GLES), 1)
	SOURCE += \
		$(SRCDIR)/Glitch64/OGLEScombiner.cpp \
		$(SRCDIR)/Glitch64/OGLESgeometry.cpp \
		$(SRCDIR)/Glitch64/OGLESglitchmain.cpp \
		$(SRCDIR)/Glitch64/OGLEStextures.cpp
else
	SOURCE += \
		$(SRCDIR)/Glitch64/OGLcombiner.cpp \
		$(SRCDIR)/Glitch64/OGLgeometry.cpp \
		$(SRCDIR)/Glitch64/OGLglitchmain.cpp \
		$(SRCDIR)/Glitch64/OGLtextures.cpp
endif

ifeq ($(USE_FRAMESKIPPER), 1)
	SOURCE += \
		$(SRCDIR)/Glide64/FrameSkipper.cpp
endif

ifneq ($(HIRES), 0)
SOURCE += \
	$(SRCDIR)/GlideHQ/Ext_TxFilter.cpp \
	$(SRCDIR)/GlideHQ/TxFilterExport.cpp \
	$(SRCDIR)/GlideHQ/TxFilter.cpp \
	$(SRCDIR)/GlideHQ/TxCache.cpp \
	$(SRCDIR)/GlideHQ/TxTexCache.cpp \
	$(SRCDIR)/GlideHQ/TxHiResCache.cpp \
	$(SRCDIR)/GlideHQ/TxQuantize.cpp \
	$(SRCDIR)/GlideHQ/TxUtil.cpp \
	$(SRCDIR)/GlideHQ/TextureFilters.cpp \
	$(SRCDIR)/GlideHQ/TextureFilters_2xsai.cpp \
	$(SRCDIR)/GlideHQ/TextureFilters_hq2x.cpp \
	$(SRCDIR)/GlideHQ/TextureFilters_hq4x.cpp \
	$(SRCDIR)/GlideHQ/TxImage.cpp \
	$(SRCDIR)/GlideHQ/TxReSample.cpp \
	$(SRCDIR)/GlideHQ/TxDbg.cpp \
	$(SRCDIR)/GlideHQ/tc-1.1+/fxt1.c \
	$(SRCDIR)/GlideHQ/tc-1.1+/wrapper.c \
	$(SRCDIR)/GlideHQ/tc-1.1+/texstore.c

CPPFLAGS += -DTEXTURE_FILTER -DDUMP_CACHE

  ifeq ($(TXCDXTN), 1)
    CPPFLAGS += -DTXCDXTN_EXTERNAL
  else 
    SOURCE += \
	$(SRCDIR)/GlideHQ/tc-1.1+/s2tc/s2tc_algorithm.cpp \
	$(SRCDIR)/GlideHQ/tc-1.1+/s2tc/s2tc_libtxc_dxtn.cpp
  endif
endif

ifeq ($(OS),MINGW)
SOURCE += \
	$(SRCDIR)/Glide64/osal_dynamiclib_win32.c \
	$(SRCDIR)/GlideHQ/osal_files_win32.c
else
SOURCE += \
	$(SRCDIR)/Glide64/osal_dynamiclib_unix.c \
	$(SRCDIR)/GlideHQ/osal_files_unix.c
endif
 


# generate a list of object files build, make a temporary directory for them
OBJECTS := $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(filter %.c, $(SOURCE)))
OBJECTS += $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(filter %.cpp, $(SOURCE)))
OBJDIRS = $(dir $(OBJECTS))
$(shell $(MKDIR) $(OBJDIRS))

# build targets
TARGET = mupen64plus-video-glide64mk2$(POSTFIX).$(SO_EXTENSION)

targets:
	@echo "Mupen64plus-video-glide64mk2 N64 Graphics plugin makefile. "
	@echo "  Targets:"
	@echo "    all           == Build Mupen64plus-video-glide64mk2 plugin"
	@echo "    clean         == remove object files"
	@echo "    rebuild       == clean and re-build all"
	@echo "    install       == Install Mupen64Plus-video-glide64mk2 plugin"
	@echo "    uninstall     == Uninstall Mupen64Plus-video-glide64mk2 plugin"
	@echo "  Options:"
	@echo "    BITS=32       == build 32-bit binaries on 64-bit machine"
	@echo "    NO_ASM=1      == build without inline assembly code (x86 MMX/SSE)"
	@echo "    USE_GLES=1    == build against GLESv2 instead of OpenGL"
	@echo "    VC=1          == build against Broadcom Videocore GLESv2"
	@echo "    NO_SSE=1      == build without SSE support"
	@echo "    USE_FRAMESKIPPER=1 == build with frameskipper feature"
	@echo "    APIDIR=path   == path to find Mupen64Plus Core headers"
	@echo "    OPTFLAGS=flag == compiler optimization (default: -O3 -flto)"
	@echo "    WARNFLAGS=flag == compiler warning levels (default: -Wall)"
	@echo "    PIC=(1|0)     == Force enable/disable of position independent code"
	@echo "    POSTFIX=name  == String added to the name of the the build (default: '')"
	@echo "    HIRES=(1|0)   == Enables/Disables support for hires textures and texture filters (default: 1)"
	@echo "    TXCDXTN=(1|0) == Enable/Disable external txc_dxtn library (default: 0)"
	@echo "  Install Options:"
	@echo "    PREFIX=path   == install/uninstall prefix (default: /usr/local)"
	@echo "    SHAREDIR=path == path to install shared data files (default: PREFIX/share/mupen64plus)"
	@echo "    LIBDIR=path   == library prefix (default: PREFIX/lib)"
	@echo "    PLUGINDIR=path == path to install plugin libraries (default: LIBDIR/mupen64plus)"
	@echo "    DESTDIR=path  == path to prepend to all installation paths (only for packagers)"
	@echo "  Debugging Options:"
	@echo "    DEBUG=1       == add debugging symbols"
	@echo "    V=1           == show verbose compiler output"

all: $(TARGET)

install: $(TARGET)
	$(INSTALL) -d "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -m 0644 $(INSTALL_STRIP_FLAG) $(TARGET) "$(DESTDIR)$(PLUGINDIR)"
	$(INSTALL) -d "$(DESTDIR)$(SHAREDIR)"
	$(INSTALL) -m 0644 "$(SRCDIR)/../data/Glide64mk2.ini" "$(DESTDIR)$(SHAREDIR)"

uninstall:
	$(RM) "$(DESTDIR)$(PLUGINDIR)/$(TARGET)"
	$(RM) "$(DESTDIR)$(SHAREDIR)/Glide64mk2.ini"

clean:
	$(RM) -r $(OBJDIR) $(TARGET)

rebuild: clean all

# build dependency files
CFLAGS += -MD -MP
-include $(OBJECTS:.o=.d)

CXXFLAGS += $(CFLAGS)

# standard build rules
$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(COMPILE.c) -o $@ $<

$(OBJDIR)/%.o: $(SRCDIR)/%.cpp
	$(COMPILE.cc) -o $@ $<

$(TARGET): $(OBJECTS)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@

.PHONY: all clean install uninstall targets
