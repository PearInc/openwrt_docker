include $(TOPDIR)/rules.mk

PKG_NAME:=docker-binary-aarch64
PKG_VERSION:=18.09.5
PKG_RELEASE:=1
PKG_FLAGS:=nonshared

PKG_BUILD_DIR:=$(BUILD_DIR)/docker
PKG_SOURCE:=docker-$(PKG_VERSION).tgz
PKG_SOURCE_URL:=https://download.docker.com/linux/static/stable/aarch64/
PKG_HASH:=549ea10709d9ed22b6435a072ea2d9dd7fc14950eb141cfbdd4653d0c12a54e2

include $(INCLUDE_DIR)/package.mk
include ../docker/docker-common.mk

define Package/docker-binary-aarch64
	$(call Package/docker-binary-common/default)
	DEPENDS+=@aarch64
endef

define Package/docker-binary-aarch64/description
	$(call Pckage/docker-binary-common/description)
endef

define Package/docker-binary-aarch64/conffiles
	$(call Package/docker-common/conffiles)
endef

define Build/Compile
	ls
endef

define Package/docker-binary-aarch64/install
	$(call Package/docker-binary-common/install,$(1))
endef

$(eval $(call BuildPackage,docker-binary-aarch64))
