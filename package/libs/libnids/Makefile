#
# Copyright (C) 2025
#
# This is free software, licensed under the GPL-2.0-only license.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=libnids
PKG_VERSION:=1.26
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/MITRECND/$(PKG_NAME)/archive/refs/tags/
PKG_HASH:=3f3e9f99a83cd37bc74af83d415c5e3a7505f5b190dfaf456b0849e0054f6733

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=COPYING
PKG_MAINTAINER:=

PKG_FIXUP:=autoreconf
PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/autotools.mk

define Package/libnids
  SECTION:=libs
  CATEGORY:=Libraries
  TITLE:=Network intrusion detection library
  URL:=https://github.com/MITRECND/libnids
  DEPENDS:=+libpcap +libnet
endef

define Package/libnids/description
 Libnids provides IP defragmentation, TCP stream assembly, and TCP scan detection.
endef

CONFIGURE_ARGS += \
	--with-libpcap=$(STAGING_DIR)/usr \
	--with-libnet=$(STAGING_DIR)/usr \
	--disable-libglib \
	--enable-shared \
	--disable-static

MAKE_FLAGS += \
	install_prefix="$(PKG_INSTALL_DIR)"

define Package/libnids/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libnids.so.* $(1)/usr/lib/
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/nids.h $(1)/usr/include/
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libnids.{a,so*} $(1)/usr/lib/
	if [ -f "$(PKG_INSTALL_DIR)/usr/lib/pkgconfig/libnids.pc" ]; then \
		$(INSTALL_DIR) $(1)/usr/lib/pkgconfig; \
		$(CP) $(PKG_INSTALL_DIR)/usr/lib/pkgconfig/libnids.pc $(1)/usr/lib/pkgconfig/; \
	fi
endef

$(eval $(call BuildPackage,libnids))
