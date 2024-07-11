#-----------------------------------------------------------------------#
#- GLOBAL DEFS ---------------------------------------------------------#
#-----------------------------------------------------------------------#

# Keep this as generic as possible.

NAME=${library.name}
VERSION=${library.version}
ARCHIVE=${library.archive.name}
URL=${library.download.url}

#-----------------------------------------------------------------------#
# Solver signatures have to be both valid file names and C symbols.
# Since Picosat uses a dash '-' for the package name, we have to
# differentiate between with (directory / package name) and without
# dash (the signature).

SIG=$(NAME)$(VERSION)$(ARCHIVE)
DIR=$(NAME)-$(ARCHIVE)
TARGET	?= dlib

#-----------------------------------------------------------------------#

# Make sets default value for CXX, we want to override them
# but not if it set by the user
# see: https://stackoverflow.com/a/42958970
ifeq ($(origin CXX),default)
export CXX		= g++
endif
export AR		?= ar
# to build universal binary on MacOS, we switch ar rc -> -static -o  
AR += rc
export CXXFLAGS	+= -Wall -DNDEBUG -O3 -fPIC -DQUIET

#-----------------------------------------------------------------------#
#- REQUIRED TOP RULES --------------------------------------------------#
#-----------------------------------------------------------------------#

all: $(TARGET)

clean:
	rm -rf $(DIR)
	rm -f *.o *.a *.tar.gz *.dylib *.dll *.so

#-----------------------------------------------------------------------#
#- INVISIBLE INTERNAL SUB RULES ----------------------------------------#
#-----------------------------------------------------------------------#

# enable verbatim output
ifeq ($(VERB),)
VB=@
else
VB=
endif

# cross-compilation
ifeq ($(CHOST),)
else
	CROSS_PREFIX = $(CHOST)-
	CXX := $(CROSS_PREFIX)$(CXX)
	AR := $(CROSS_PREFIX)$(AR)
	UNAME = $(shell echo "$(CHOST)" | sed -e 's/^[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)-.*$$/\1/')
endif

# determine operating system name
UNAME ?= $(shell uname -s)

# the location of this makefile (independent of the current directory)
MROOT := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))

# the built glue object file implementing the ipasir interface
IPASIRGLUEOBJ=$(DIR)/build/src/ipasir.o
# the static library of the solver, including the ipasir code
STATICLIB=$(DIR)/build/libcadical.a

ifeq ($(UNAME), Darwin)
 	DLIB = lib$(NAME).dylib
 	# build universal "fat" binaries
 	CXXFLAGS += -arch x86_64 -arch arm64
 	AR = libtool -static -o
else ifeq ($(UNAME), Linux)
 	DLIB = lib$(NAME).so
else ifeq ($(UNAME), mingw32)
 	DLIB = $(NAME).dll
	# TODO: move these options to LDFLAGS once they are supported
	# https://github.com/arminbiere/cadical/pull/88
	CXXFLAGS += -static-libgcc -static-libstdc++
	LIBS += -lpsapi
else
 	$(error Platform not supported: $(UNAME))
endif 

dlib: $(DLIB)

# shared library for Mac OS
lib$(NAME).dylib: $(IPASIRGLUEOBJ) $(STATICLIB)
	$(VB) $(CXX) $(CXXFLAGS) -dynamiclib -o $@ $^ -install_name $@ $(LIBS)

# shared library for Linux
lib$(NAME).so: $(IPASIRGLUEOBJ) $(STATICLIB)
	$(VB) $(CXX) $(CXXFLAGS) -shared -o $@ $^ -Wl,-soname,$@ $(LIBS)

# shared library for MinGW
$(NAME).dll: $(IPASIRGLUEOBJ) $(STATICLIB)
	$(VB) $(CXX) $(CXXFLAGS) -shared -o $@ $^ -Wl,--subsystem,windows $(LIBS)

# static solver library
$(IPASIRGLUEOBJ) $(STATICLIB): $(DIR)
	@#
	@# configure and build library
	@#
	# patch to use a configurable archive tool
	$(VB) sed -i.bak -e's/ar rc/$(AR)/' $(DIR)/makefile.in
	$(VB) cd $(DIR); ./configure $(LIBS) && make cadical

$(DIR): $(ARCHIVE).tar.gz
	@#
	@# extract library
	@#
	$(VB) tar xf $<

$(ARCHIVE).tar.gz:
	@#
	@# download library
	@#
	curl -L "$(URL)$@" --output $@

#-----------------------------------------------------------------------#

.FORCE:
.PHONY: all clean