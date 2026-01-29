#
# Copyright (C) 2025
#
# This is free software, licensed under the BSD-3-Clause license.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=dsniff
PKG_VERSION:=2.3
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://www.monkey.org/~dugsong/$(PKG_NAME)/
PKG_HASH:=82e492455486e655c315f027d393dbeb49ad930804acccdc51b30d57e1294ff5

PKG_LICENSE:=BSD-3-Clause
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=

PKG_FIXUP:=autoreconf

include $(INCLUDE_DIR)/package.mk

define Package/dsniff
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Network auditing and penetration testing tools
  URL:=https://www.monkey.org/~dugsong/dsniff/
  DEPENDS:=+libpcap +libopenssl +libdb47 +libnet +libnids +libtirpc
endef

define Package/dsniff/description
 dsniff is a collection of tools for network auditing and penetration testing.
endef

CONFIGURE_ARGS += \
	--with-libpcap=$(STAGING_DIR)/usr \
	--with-libnet=$(STAGING_DIR)/usr \
	--with-libnids=$(STAGING_DIR)/usr \
	--with-openssl=$(STAGING_DIR)/usr \
	--with-db=$(STAGING_DIR)/usr \
	--libdir=/usr/lib/dsniff \
	--sbindir=/usr/sbin \
	--without-x

CONFIGURE_VARS += \
	PATH="$(STAGING_DIR_TOOLCHAIN)/bin:$(STAGING_DIR_HOST)/bin:/usr/bin:/bin:$(STAGING_DIR)/usr/bin" \
	CC="$(TOOLCHAIN_ROOT_DIR)/bin/$(TARGET_CROSS)gcc" \
	CXX="$(TOOLCHAIN_ROOT_DIR)/bin/$(TARGET_CROSS)g++" \
	ac_cv_path_libnet_config="$(STAGING_DIR)/usr/bin/libnet-config" \
	CFLAGS="$(TARGET_CFLAGS) -I$(STAGING_DIR)/usr/include/tirpc" \
	CPPFLAGS="$(TARGET_CPPFLAGS) -I$(STAGING_DIR)/usr/include/tirpc"

TARGET_LDFLAGS += -ltirpc

define Package/dsniff/install
	$(INSTALL_DIR) $(1)/usr/sbin
	for f in arpspoof dnsspoof dsniff filesnarf macof mailsnarf msgsnarf \
		tcpkill tcpnice urlsnarf webmitm; do \
		if [ -f "$(PKG_BUILD_DIR)/$$$$f" ]; then \
			$(INSTALL_BIN) "$(PKG_BUILD_DIR)/$$$$f" "$(1)/usr/sbin/"; \
		fi; \
	done
	$(INSTALL_DIR) $(1)/usr/lib/dsniff
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/dsniff.magic $(1)/usr/lib/dsniff/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/dsniff.services $(1)/usr/lib/dsniff/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/dnsspoof.hosts $(1)/usr/lib/dsniff/
endef

$(eval $(call BuildPackage,dsniff))
