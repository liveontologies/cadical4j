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

# the location of this makefile (independent of the current directory)
MROOT := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))

# cross-compilation
ifneq ($(CHOST),)
	CROSS_PREFIX = $(CHOST)-
	CXX := $(CROSS_PREFIX)$(CXX)
	# prefer -posix version if there are any
	ifneq ($(shell command -v "$(CXX)-posix"),)
		CXX := $(CXX)-posix 
	endif	
	AR := $(CROSS_PREFIX)$(AR)
	export SKIP_EXEC=1
	UNAME := $(shell echo "$(CHOST)" | sed -e 's/^[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)-.*$$/\1/')#						
endif

# determine operating system name
UNAME ?= $(shell uname -s)

# the build glue object file implementing the ipasir interface
IPASIRGLUEOBJ=$(DIR)/build/ipasir.o
# the static library of the solver, including the ipasir code
STATICLIB=$(DIR)/build/libcadical.a

ifeq ($(UNAME), Darwin)
 	DLIB = lib$(NAME).dylib
 	# build universal "fat" binaries
 	CFLAGS += -arch x86_64 -arch arm64
 	CXXFLAGS += -arch x86_64 -arch arm64
 	AR = libtool -static -o
else ifeq ($(UNAME), Linux)
 	DLIB = lib$(NAME).so
else ifeq ($(UNAME), mingw32)
 	DLIB = $(NAME).dll
 	CXXFLAGS := $(CXXFLAGS) -D_POSIX -D__USE_MINGW_ALARM 
else
 	$(error Platform not supported: $(UNAME))
endif 

dlib: $(DLIB)

# shared library for Mac OS
lib$(NAME).dylib: $(STATICLIB)
	$(VB) $(CXX) $(CFLAGS) -g -I. -o $@ $(IPASIRGLUEOBJ) $^ -dynamiclib -install_name $@ 

# shared library for Linux
lib$(NAME).so: $(STATICLIB)
	$(VB) $(CXX) -o $@ $(IPASIRGLUEOBJ) $^ $(CFLAGS) -g -I. -shared -Wl,-soname,$@ 

# shared library for MinGW
$(NAME).dll: $(STATICLIB)
	$(VB) $(CXX) -o $@ $(IPASIRGLUEOBJ) $^ $(CFLAGS) -static-libgcc -static-libstdc++ -g -I. -shared -Wl,--subsystem,windows -Wl,-Bstatic -lstdc++ -lpthread

# static solver library
$(STATICLIB): $(DIR)
	@#
	@# configure and build library
	@#
	# patch to use a configurable archive tool
	$(VB) sed -i.bak -e's/$$(AR) rc/$$(AR)/' $(DIR)/makefile.in
	$(VB) cd $(DIR); ./configure && make cadical	


$(DIR): $(ARCHIVE).tar.gz
	@#
	@# extract library
	@#
	$(VB) tar xf $<

$(ARCHIVE).tar.gz:
	@#
	@# download library
	@#
	wget "$(URL)$@"

#-----------------------------------------------------------------------#

.FORCE:
.PHONY: all clean
