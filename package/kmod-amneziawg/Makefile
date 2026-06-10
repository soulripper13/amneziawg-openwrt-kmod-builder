include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=amneziawg
PKG_VERSION:=1.0.0
PKG_RELEASE:=1
PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=COPYING

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define KernelPackage/amneziawg
  SUBMENU:=Network Support
  TITLE:=AmneziaWG VPN Kernel Module
  DEPENDS:=+kmod-udptunnel4 +kmod-udptunnel6 +kmod-crypto-lib-chacha20poly1305 +kmod-crypto-lib-curve25519
  FILES:=$(PKG_BUILD_DIR)/src/amneziawg.ko
  AUTOLOAD:=$(call AutoProbe,amneziawg)
endef

define KernelPackage/amneziawg/description
  AmneziaWG VPN kernel module.
endef

define Build/Prepare
	$(RM) -r $(PKG_BUILD_DIR)
	$(INSTALL_DIR) $(PKG_BUILD_DIR)
	$(CP) ./src $(PKG_BUILD_DIR)/
	$(CP) ./src/COPYING $(PKG_BUILD_DIR)/COPYING
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)/src" WIREGUARD_VERSION="$(PKG_VERSION)" modules
endef

$(eval $(call KernelPackage,amneziawg))
