include $(TOPDIR)/rules.mk

PKG_NAME:=docker-binary-x86_64
PKG_VERSION:=18.09.5
PKG_RELEASE:=1
PKG_FLAGS:=nonshared

PKG_BUILD_DIR:=$(BUILD_DIR)/docker
PKG_SOURCE:=docker-$(PKG_VERSION).tgz
PKG_SOURCE_URL:=https://download.docker.com/linux/static/stable/x86_64/
PKG_HASH:=99ca9395e9c7ffbf75537de71aa828761f492491d02bc6e29db2920fa582c6c5

include $(INCLUDE_DIR)/package.mk
include ../docker/docker-common.mk

define Package/docker-binary-x86_64
	$(call Package/docker-binary-common/default)
	DEPENDS+=@TARGET_x86_64
endef

define Package/docker-binary-x86_64/description
 Docker daemon (pre-compiled static binary)
endef

define Build/Compile
	ls
endef

define Package/docker-binary-x86_64/install
	$(call Package/docker-binary-common/install,$(1))
endef

$(eval $(call BuildPackage,docker-binary-x86_64))
