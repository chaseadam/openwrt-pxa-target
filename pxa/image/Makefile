#
# Copyright (C) 2008-2012 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk
include $(INCLUDE_DIR)/host.mk

SQUASHFSCOMP:=xz
JFFS2_BLOCKSIZE:=64k

define Image/BuildKernel
	cat $(KDIR)/zImage $(DTS_DIR)/zipit-z2.dtb > $(KDIR_TMP)/zImage-dtb 
	mkimage -A Arm \
		-O linux \
		-C none \
		-T kernel \
		-a 0xA0008000 -e 0xA0008000 \
		-n uImage-dtb -d $(KDIR_TMP)/zImage-dtb $(KDIR_TMP)/uImage
	cp $(KDIR_TMP)/uImage $(BIN_DIR)/$(IMG_PREFIX)-uImage
endef

# Build a fake flash image for usage in Qemu
define Image/Build/Gumstix
	dd if=/dev/zero bs=128k count=256 of=$(BIN_DIR)/$(IMG_PREFIX)-$(1)-qemu-flash.img
	dd if=$(BIN_DIR)/openwrt-pxa-gumstix-u-boot.bin conv=notrunc bs=128k \
		of=$(BIN_DIR)/$(IMG_PREFIX)-$(1)-qemu-flash.img
	dd if=$(KDIR)/root.$(1) conv=notrunc bs=128k seek=2 \
		of=$(BIN_DIR)/$(IMG_PREFIX)-$(1)-qemu-flash.img
	dd if=$(KDIR)/uImage conv=notrunc bs=128k seek=248 \
		of=$(BIN_DIR)/$(IMG_PREFIX)-$(1)-qemu-flash.img
endef

define Image/Build
	$(call Image/Build/$(1),$(1))
	$(call Image/Build/$(PROFILE),$(1))
endef

define Image/Build/jffs2-64k
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/$(IMG_PREFIX)-$(1).img bs=64k conv=sync
endef

define Image/Build/squashfs
	$(call prepare_generic_squashfs,$(KDIR)/root.squashfs)
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/$(IMG_PREFIX)-$(1).img bs=64k conv=sync
	mkdir -p $(KDIR_TMP)/empty_jffs
	$(STAGING_DIR_HOST)/bin/mkfs.jffs2  -v -X lzma -y 80:lzma -y 70:zlib \
		--pad=2162688 --little-endian --squash -e 64KiB \
		-d '$(KDIR_TMP)/empty_jffs' \
		-o '$(BIN_DIR)/$(IMG_PREFIX)-overlay.jffs2'  
endef

$(eval $(call BuildImage))
