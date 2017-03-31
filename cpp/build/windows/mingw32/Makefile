#
# Makefile for OpenZWave MingW32 build
# Maarten Damen

.SUFFIXES:	.d .cpp .o .a
.PHONY:	default clean

CC     := $(CROSS_COMPILE)gcc
CXX    := $(CROSS_COMPILE)g++
LD     := $(CROSS_COMPILE)g++
AR     := $(CROSS_COMPILE)ar rc
RANLIB := $(CROSS_COMPILE)ranlib

VERSION_MAJ	?= 1
VERSION_MIN	?= 4
VERSION_REV	?= 0
GITVERSION	:= $(VERSION_MAJ).$(VERSION_MIN).-1

DEBUG_CFLAGS    := -DDEBUG -DLOG_STDERR
RELEASE_CFLAGS  := -O3

DEBUG_LDFLAGS	:= -g

# Change for DEBUG or RELEASE
CFLAGS	:= -c -Wall -Wno-unknown-pragmas -Wno-format -DMINGW $(DEBUG_CFLAGS)
LDFLAGS	:= $(DEBUG_LDFLAGS)
LDLIBS	:= -lsetupapi

LIBDIR	:= ../../../lib/windows-mingw32

INCLUDES	:= -I ../../../src -I ../../../src/command_classes/ -I ../../../src/value_classes/ \
	-I ../../../src/platform/ -I ../../../src/platform/windows -I ../../../tinyxml/ -I ../../../hidapi/hidapi/
SOURCES		:= ../../../src ../../../src/command_classes ../../../tinyxml ../../../hidapi/windows \
	../../../src/value_classes ../../../src/platform ../../../src/platform/windows \
	../../../src/aes

VPATH = ../../../src:../../../src/command_classes:../../../tinyxml:../../../hidapi/windows:\
	../../../src/value_classes:../../../src/platform:../../../src/platform/windows \
	../../../src/aes

%.d : %.cpp
	@set -e; rm -f $@; \
	$(CXX) -MM $(INCLUDES) $< > $@.$$$$; \
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	rm -f $@.$$$$

tinyxml := $(notdir $(wildcard ../../../tinyxml/*.cpp))
hidapi := $(notdir $(wildcard ../../../hidapi/linux/hid.c)) # we do not want the libusb version
cclasses := $(notdir $(wildcard ../../../src/command_classes/*.cpp))
vclasses := $(notdir $(wildcard ../../../src/value_classes/*.cpp))
aes := $(notdir $(wildcard ../../../src/aes/*.c))
pform := $(notdir $(wildcard ../../../src/platform/*.cpp)) \
	$(notdir $(wildcard ../../../src/platform/windows/*.cpp))
indep := $(notdir $(wildcard ../../../src/*.cpp))

%.o : %.cpp
	$(CXX) $(CFLAGS) $(INCLUDES) -o $@ $<

%.o : %.c
	$(CC) $(CFLAGS) $(INCLUDES) -o $@ $<

default:	$(LIBDIR)/openzwave.a \
		$(LIBDIR)/openzwave.dll

clean:
	rm -f *.d *.o

-include $(tinyxml:.cpp=.d)
-include $(hidapi:.c=.d)
-include $(cclasses:.cpp=.d)
-include $(vclasses:.cpp=.d)
-include $(aes:.c=.d)
-include $(pform:.cpp=.d)
-include $(indep:.cpp=.d)

vers.c:
	echo '#include <stdint.h>' > $@
	echo 'uint16_t ozw_vers_major = $(VERSION_MAJ);' >> $@
	echo 'uint16_t ozw_vers_minor = $(VERSION_MIN);' >> $@
	echo 'uint16_t ozw_vers_revision = $(VERSION_REV);' >> $@
	echo 'char ozw_version_string[] = "$(GITVERSION)";' >> $@

vers.o:	vers.c

$(LIBDIR)/openzwave.a:	$(patsubst %.cpp,%.o,$(tinyxml)) \
			$(patsubst %.c,%.o,$(hidapi)) \
			$(patsubst %.cpp,%.o,$(cclasses)) \
			$(patsubst %.cpp,%.o,$(vclasses)) \
			$(patsubst %.cpp,%.o,$(pform)) \
			$(patsubst %.c,%.o,$(aes)) \
			$(patsubst %.cpp,%.o,$(indep)) vers.o \
			| $(LIBDIR)/
	$(AR) $@ $?

$(LIBDIR)/openzwave.dll: $(patsubst %.cpp,%.o,$(tinyxml)) \
			$(patsubst %.c,%.o,$(hidapi)) \
			$(patsubst %.cpp,%.o,$(cclasses)) \
			$(patsubst %.cpp,%.o,$(vclasses)) \
			$(patsubst %.cpp,%.o,$(pform)) \
			$(patsubst %.c,%.o,$(aes)) \
			$(patsubst %.cpp,%.o,$(indep)) vers.o \
			| $(LIBDIR)/
	$(LD) $(LDFLAGS) -shared -o $@ $? $(LDLIBS)

$(LIBDIR)/:
	mkdir -p $@
