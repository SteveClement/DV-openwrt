include $(TOPDIR)/rules.mk

PKG_NAME:=bettercap
PKG_VERSION:=2.41.5
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/bettercap/bettercap/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=85513871e105a182eb92f80ba9563ac37cb8a48bcfa98d30e1197e74c42ff15a

PKG_LICENSE:=GPL-3.0-or-later
PKG_LICENSE_FILES:=LICENSE.md

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/bettercap/bettercap/v2
GO_PKG_BUILD_PKG:=github.com/bettercap/bettercap/v2
GO_PKG_INSTALL_ALL:=1

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/bettercap
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Swiss army knife for network attacks and monitoring
  URL:=https://www.bettercap.org/
  DEPENDS:=$(GO_ARCH_DEPENDS) +libpcap +libnetfilter-queue +libusb-1.0 +bluez-libs
endef

define Package/bettercap/description
  bettercap is a powerful, flexible, and portable tool for network
  attacks and monitoring, supporting Wi-Fi, BLE, HID, and NFQUEUE
  packet proxying modules.
endef

define Package/bettercap/install
	$(call GoPackage/Package/Install/Bin,$(1))
	$(INSTALL_DIR) $(1)/usr/share/bettercap/caplets
endef

$(eval $(call BuildPackage,bettercap))
