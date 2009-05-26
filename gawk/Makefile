#############################################################################
#
# To build gawk on a particular build host/platform, do the following:
#
#  - Ensure that the GNU make is in your PATH (usually, that means having
#    /usr/local/bin in front of your PATH.
#
#  - Type:  make all test install
#
#  - In order to run "make install", you will likely need to check out for
#    edit the gawk binary from perforce first.  If you don't care to install
#    the gawk binary into the proper perforce directory, then skip this step
#    and instead type  "make all test".
#
#  - If the C compiler (gcc) or another tool isn't found, then you will need
#    to create/edit a small external makefile, called mklib/$HOST.mk and/or
#    mklib/$PLAT.mk, where you can define/override CONFIG_ARGS, GUNZIP, etc.
#
#
# To define/edit/upgrade a new distribution, do the following:
#
#  - Download/copy the distrib to the "dist" subdirectory.  It should be
#    saved as a compressed (gzipped) tar file.
#
#  - Set SWDIST to the new software distribution name.  Confirm that
#    SWDIST_FILE matches.
#
#  - Check if the existing patch files (in the "patches" subdirectory) are
#    reasonable for the new distrib.  You will probably need to copy the
#    patch files from the previous release subdir of the "patches" directory
#    into a new subdir that you should name to match the new release (SWDIST).
#    Type "make patch" to extract and patch the distrib.  The source tree will
#    be found in "build/src".
#
#  - Continue as above.
#
# If you make src modifications, you will need to update the patch files.
# To generate the complete set of patch files, type "make patches".  This
# will automatically compare the pristine s/w distribution with the contents
# of the "build/src" tree and produce a new set of patch files in
# "build/patches".  Manually copy these files to the PATCHDIR and check
# updates into perforce as appropriate.
#
#############################################################################


SWDIST		= gawk-3.1.5

PRODUCT		= gawk
SWDIST_FILE	= dist/$(SWDIST).tar.gz

PATCHDIR	= patches/$(SWDIST)
BUILD_BASE	= build
MKLIB		= mklib
STAGE_DIR	= $(BUILD_BASE)/.stages
SRC_UP		= $(BUILD_BASE)/src
SRC		= $(SRC_UP)/$(SWDIST)
REL_SRC		= ../../$(SRC)
NEW_PATCHES	= $(BUILD_BASE)/patches
HOST		= $(shell hostname | sed 's/\..*//')
CONFIG_ARGS	= CC=/usr/local/gcc-3.4/bin/gcc-3.4

# Work out build platform

SYS		:= $(shell uname -s)
REL		:= $(shell uname -r)

ifeq ($(SYS), AIX)
    REL1	:= $(shell uname -v)
    REL2	:= $(REL)
    REL		:= $(REL1).$(REL2)
    ifeq ($(REL1), 4)
	PLAT	= aix_4.3
    else
	PLAT	= aix_5.1
    endif
    WRONG_PLATFORM := $(filter-out 5.1 4.3,$(REL))
else
ifeq ($(SYS), HP-UX)
    REL		:= $(subst B.,,$(REL))
    ifeq ($(REL), 11.23)
	PLAT	= hpux_11.23
    else
	PLAT	= hpux_11.11
    endif
    WRONG_PLATFORM := $(filter-out 11.11 11.23,$(REL))
else
ifeq ($(SYS), Linux)
    REL		:= $(shell awk '$$1 == "Red" {print $$5 == "ES" ? "e" $$7 : $$5}' /etc/redhat-release)
    ifeq ($(REL), e2.1)
	PLAT	= linux_e2.1
    else
	PLAT	= linux_e3
    endif
    WRONG_PLATFORM := $(filter-out e3 e2.1,$(REL))
else
ifeq ($(SYS), SunOS)
    ARCH	:= $(shell uname -m)
    ifeq ($(ARCH), i86pc)
    	PLAT = solx86_5.10
    else
    	PLAT = solaris_5.6
    endif
    WRONG_PLATFORM := $(filter-out 5.6 5.10,$(REL))
else
ifeq ($(SYS), Darwin)
    ARCH := $(shell uname -p)
    PLAT = macosx10
    CONFIG_ARGS	= CC=/usr/bin/gcc
    CFLAGS="-arch i386 -arch ppc -mmacosx-version-min=10.4"
    LDFLAGS="-arch i386 -arch ppc -mmacosx-version-min=10.4"
else
    $(error Unexpected build platform:  $(SYS))
endif
endif
endif
endif
endif

ifneq ($(strip $(WRONG_PLATFORM)),)
    $(warning $(HOST) is a(n) $(SYS) $(REL) system and not the expected build \
	    platform ($(PLAT)).  Build will continue.  However, "make install"\
	    will error, and please do not manually circumvent.)
endif


BUILD		= $(BUILD_BASE)/$(HOST)
INSTDIR		= ../../bin/$(PLAT)
INST_PRODUCT	= $(INSTDIR)/$(PRODUCT)

STAGES		= extract patch config configure build test check install clean

PATCHES		= $(basename $(notdir $(wildcard $(PATCHDIR)/*.patch)))
PATCH_STAGE	= $(addprefix $(STAGE_DIR)/patch-, $(PATCHES))

CONFIG_STAGE	= $(STAGE_DIR)/configure-$(PLAT)

INST_PREFIX	= `pwd`/install_root

GUNZIP		= gunzip

# Set any platform-specific or build-host-specific variables
# (ie. set CONFIG_ARGS)
-include $(MKLIB)/$(PLAT).mk
-include $(MKLIB)/$(HOST).mk


###
### Begin Build Rules
###

all:	build

$(STAGE_DIR) $(SRC_UP) $(BUILD) $(NEW_PATCHES):
	mkdir -p $@


# (1)  Extract source distribution

extract: $(STAGE_DIR)/extract
$(STAGE_DIR)/extract:
	-@mkdir -p $(SRC_UP) $(STAGE_DIR)
	$(GUNZIP) < $(SWDIST_FILE) | (cd $(SRC_UP) && tar xf -)
	touch $@


# (2)  Apply patch(es)

patch: $(PATCH_STAGE)
$(PATCH_STAGE): $(STAGE_DIR)/patch-%: $(STAGE_DIR)/extract
$(PATCH_STAGE): $(STAGE_DIR)/patch-%: $(PATCHDIR)/%.patch
	@if [ -f $@ ]; then patch -R -d $(SRC) < $@; fi
	patch -d $(SRC) < $<
	cp $< $@


# (3)  Configure build (only needs to run once until next clean build)

config configure: $(PATCH_STAGE) $(CONFIG_STAGE)
$(CONFIG_STAGE): $(SRC)/configure
	-@mkdir -p $(BUILD)
	cd $(BUILD) && LDFLAGS=${LDFLAGS}; export LDFLAGS && CFLAGS=${CFLAGS}; export CFLAGS && $(REL_SRC)/configure --prefix=$(INST_PREFIX) $(CONFIG_ARGS)
	touch $@


# (4)  Build product (run make)

build: $(STAGE_DIR)/build
$(STAGE_DIR)/build: $(PATCH_STAGE) $(CONFIG_STAGE)
	cd $(BUILD) && $(MAKE)


# (5)  Run tests

test check: build
	cd $(BUILD) && $(MAKE) check
	@echo "BUILD NOTE:  (only) three tests are expected to fail:"
	@echo "		uninit3, uninit4, and shadow."
	@echo "	This is okay.  Anything else isn't okay."


# (6*)  Do "stage" install (*not needed: installing from build area instead)

sinstall: $(STAGE_DIR)/install
$(STAGE_DIR)/install:
	cd $(BUILD) && $(MAKE) install


# (7)  Install binary (this will fail if target isn't checked out in perforce)

install:  $(INST_PRODUCT)
$(INST_PRODUCT): build $(BUILD)/$(PRODUCT)
    ifneq ($(strip $(WRONG_PLATFORM)),)
	$(error $(HOST) is a $(SYS) $(REL) system and not the expected build \
		platform ($(PLAT)).  Do not install this product version)
    else
	@if [ ! -w $@ ]; then echo "Can't install $@:  not checked out?"; false; fi
	cp $(BUILD)/$(PRODUCT) $@
	strip $@
    endif


patches: $(NEW_PATCHES)
	-@rm -rf $(NEW_PATCHES)/$(SWDIST) $(NEW_PATCHES)/*.patch
	$(GUNZIP) < $(SWDIST_FILE) | (cd $(NEW_PATCHES) && tar xf -)
	@echo "Performing diff of src distribution with $(SRC) tree"
	@diff -c -r $(NEW_PATCHES)/$(SWDIST) $(SRC) | \
	    gawk -v pdir=$(NEW_PATCHES) ' \
		BEGIN		{ file = ""; start = 0 } \
		/^diff -c -r /	{ file = $$NF; sub(/([^\/]*\/)*/, "", file); \
				  file = file ".patch"; start = NR; next } \
		NR-start < 3 && ($$1 == "---" || $$1 == "***") { \
				  sub(/([^\/]*\/)*/, "", $$2); \
				  if (NR-start == 2) $$2 = $$2 ".new" \
				} \
		file		{ print > pdir "/" file } \
	    ' || true
	-@rm -rf $(NEW_PATCHES)/$(SWDIST)
	@echo "Patch files saved in $(NEW_PATCHES)"

# Clean

clean:
	rm -rf $(BUILD_BASE)


.PHONY: $(STAGES) patches
