# Default target is omap3evm/AM37xevm
TARGET_PRODUCT ?= omap3evm

# Use rowboat kernel to build against
KLIB ?= $(ANDROID_ROOT_DIR)/kernel
KLIB_BUILD := $(KLIB)

# Use android toolchain
CROSS_COMPILE=$(ANDROID_ROOT_DIR)/$(TOOLS_PREFIX)

# Install the compat drivers to the specified $TARGET_PRODUCT only
DESTDIR?=$(ANDROID_ROOT_DIR)/out/target/product/$(TARGET_PRODUCT)

ifneq ($(KERNELRELEASE),)

-include $(COMPAT_CONFIG)
include $(COMPAT_CONFIG_CW)

NOSTDINC_FLAGS := -I$(M)/include/ \
	-include $(M)/include/linux/compat-2.6.h \
	$(CFLAGS)

obj-y := compat/


ifeq ($(BT),)
obj-$(CONFIG_COMPAT_WIRELESS) += net/wireless/ net/mac80211/
obj-$(CONFIG_COMPAT_WIRELESS_MODULES) += drivers/net/wireless/

endif
else

export PWD :=	$(shell pwd)

# The build will fail if there is any space in PWD.
ifneq (,$(findstring  $() ,$(PWD)))
$(error "The path to this compat-wireless directory has spaces in it." \
	"Please put it somewhere where there is no space")
endif

export CFLAGS += \
        -DCOMPAT_BASE="\"$(shell cat $(PWD)/.compat_base)\"" \
        -DCOMPAT_BASE_TREE="\"$(shell cat $(PWD)/.compat_base_tree)\"" \
        -DCOMPAT_BASE_TREE_VERSION="\"$(shell cat $(PWD)/.compat_base_tree_version)\"" \
        -DCOMPAT_PROJECT="\"Compat-wireless\"" \
        -DCOMPAT_VERSION="\"$(shell cat $(PWD)/.compat_version)\""

# These exported as they are used by the scripts
# to check config and compat autoconf
export COMPAT_CONFIG_CW=$(PWD)/config.mk
export COMPAT_CONFIG=$(PWD)/.config
export CONFIG_CHECK=$(PWD)/.config.mk_md5sum.txt
export COMPAT_AUTOCONF=include/linux/compat_autoconf.h
export CREL=$(shell cat $(PWD)/.compat_version)
export CREL_PRE:=.compat_autoconf_
export CREL_CHECK:=$(PWD)/$(CREL_PRE)$(CREL)

all: modules

$(COMPAT_CONFIG): ;

modules: $(CREL_CHECK)
	$(MAKE) -C $(KLIB_BUILD) M=$(PWD) modules
	@touch $@

	@./scripts/check_config.sh
	$(MAKE) -j1 -C $(KLIB_BUILD) M=$(PWD) modules
	@touch $@

# We use a CREL_CHECK variable which will depend on the environment used to
# build. If the environment requirements change it forces a reconfiguration
# check.  This means we force a new reconfiguration check if a the user gets a
# new updates of compat-wireless or when the user updates the $(COMPAT_CONFIG)
# file.
# XXX: add kernel target to the CREL_CHECK mix, this would ensure we also
# reconfigure and build again fresh if we detect a new target kernel is
# being used.
$(CREL_CHECK):
	@# Force to regenerate compat autoconf
	+@./compat/scripts/gen-compat-config.sh > $(COMPAT_CONFIG)
	@rm -f $(CONFIG_CHECK)
	+@./scripts/check_config.sh
	@md5sum $(COMPAT_CONFIG_CW) > $(CONFIG_CHECK)
	@touch $@

install: modules
	@echo "Installing WL12xx compat driver..."
	@mkdir -p $(DESTDIR)/system/lib/modules
	@install compat/compat.ko                              $(DESTDIR)/system/lib/modules
	@install drivers/net/wireless/ti/wl12xx/wl12xx.ko      $(DESTDIR)/system/lib/modules
	@install drivers/net/wireless/ti/wl18xx/wl18xx.ko      $(DESTDIR)/system/lib/modules
	@install net/mac80211/mac80211.ko                      $(DESTDIR)/system/lib/modules
	@install net/wireless/cfg80211.ko                      $(DESTDIR)/system/lib/modules
	@install drivers/net/wireless/ti/wlcore/wlcore.ko      $(DESTDIR)/system/lib/modules
	@install drivers/net/wireless/ti/wlcore/wlcore_sdio.ko $(DESTDIR)/system/lib/modules

clean:
	@if [ -d net -a -d $(KLIB_BUILD) ]; then \
		$(MAKE) -C $(KLIB_BUILD) M=$(PWD) clean ;\
	fi
	@rm -f $(CREL_PRE)*

.PHONY: all clean install modules

endif
