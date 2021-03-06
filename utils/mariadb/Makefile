#
# Copyright (C) 2006-2015 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/uclibc++.mk

PKG_NAME:=mariadb
PKG_VERSION:=10.2.7
PKG_RELEASE:=2

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=\
  http://mirror.nodesdirect.com/mariadb//mariadb-$(PKG_VERSION)/source/

PKG_MD5SUM:=81727986035c5835e0fbfdf99af71428
PKG_MAINTAINER:=CZ.NIC <packaging@turris.cz>
PKG_LICENSE:=GPL-2.0

PKG_BUILD_DEPENDS:=libncurses libreadline mariadb/host
PKG_BUILD_PARALLEL:=1

HOST_BUILD_DEPENDS:=ncurses/host openssl/host pkg-config

PKG_FIXUP:=libtool

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/cmake.mk

CMAKE_OPTIONS:= -DSTACK_DIRECTION=-1 \
                -DHAVE_IB_GCC_ATOMIC_BUILTINS=1 \
                -DIMPORT_EXECUTABLES=$(STAGING_DIR_HOST)/mariadb-import_executables \
                -DPLUGIN_HANDLERSOCKET=NO \
                -DPLUGIN_XTRADB=DYNAMIC \
                -DPLUGIN_TOKUDB=DYNAMIC \
                -DWITH_UNIT_TESTS=OFF \
                -DWITH_WSREP=OFF \
                -DWITH_INNODB_LZO=OFF \
                -DWITH_INNODB_BZIP2=OFF \
                -DINSTALL_MYSQLSHAREDIR=share/mariadb \
                -DINSTALL_LAYOUT=RPM \
                -DINSTALL_PLUGINDIR=lib/mariadb/plugin \
                -DINSTALL_MYSQLDATADIR=/srv/mysql \
                -DINSTALL_UNIX_ADDRDIR=/var/run/mysql.sock

define Package/libmariadbclient
  SECTION:=libs
  CATEGORY:=Libraries
  DEPENDS:=$(CXX_DEPENDS) +zlib +libpthread +libopenssl
  TITLE:=MariaDB client library
  URL:=http://dev.mysql.com/
endef

define Package/mariadb-server
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+libmariadbclient +libpthread +libncurses +libreadline +zlib +libpcre +libstdcpp +libxml2 +libpam +libedit +liblzma +libaio +liblz4 +unixodbc
  TITLE:=MariaDB Server
  URL:=http://dev.mysql.com/
  SUBMENU:=database
  USERID:=mysql=6446:mysql=6446
endef

define Host/Configure
	cd $(HOST_BUILD_DIR); \
	mkdir build; \
	cd build; \
	export PKG_CONFIG_LIBDIR=$${PKG_CONFIG_LIBDIR}:$(STAGING_DIR_HOST)/openssl-host/lib/pkgconfig ;\
	export PKG_CONFIG_PATH=$${PKG_CONFIG_PATH}:$(STAGING_DIR_HOST)/openssl-host/lib/pkgconfig ;\
	cmake \
                       -DCMAKE_BUILD_TYPE=Release \
                        -DCMAKE_C_FLAGS_RELEASE="-DNDEBUG" \
                        -DCMAKE_CXX_FLAGS_RELEASE="-DNDEBUG" \
                        -DCMAKE_EXE_LINKER_FLAGS:STRING="$(HOST_LDFLAGS)" \
                        -DCMAKE_MODULE_LINKER_FLAGS:STRING="$(HOST_LDFLAGS)" \
                        -DCMAKE_SHARED_LINKER_FLAGS:STRING="$(HOST_LDFLAGS)" \
                        -DCMAKE_FIND_ROOT_PATH="$(CMAKE_HOST_FIND_ROOT_PATH)" \
                        -DCMAKE_FIND_ROOT_PATH_MODE_PROGRAM=BOTH \
                        -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY \
                        -DCMAKE_FIND_ROOT_PATH_MODE_INCLUDE=ONLY \
                        -DCMAKE_STRIP=: \
                        -DCMAKE_INSTALL_PREFIX=$(STAGING_DIR_HOST) \
                        -DCMAKE_PREFIX_PATH=$(STAGING_DIR_HOST) \
			-DWITH_SSL=system \
			-DWITH_SSL=$(STAGING_DIR_HOST)/openssl-host \
			-DWITH_SSL_PATH=$(STAGING_DIR_HOST)/openssl-host \
			-DOPENSSL_FOUND=TRUE \
			-DOPENSSL_ROOT_DIR=$(STAGING_DIR_HOST)/openssl-host \
			-DOPENSSL_INCLUDE_DIR=$(STAGING_DIR_HOST)/openssl-host/include \
			-DOPENSSL_SSL_LIBRARY=$(STAGING_DIR_HOST)/openssl-host/lib/libssl.so \
			-DOPENSSL_CRYPTO_LIBRARY=$(STAGING_DIR_HOST)/openssl-host/lib/libcrypto.so \
                        -DCMAKE_SKIP_RPATH=TRUE  \
			-DCURSES_INCLUDE_PATH=$(STAGING_DIR_HOST)/usr/include \
			-DCURSES_LIBRARY=$(STAGING_DIR_HOST)/usr/lib \
                	-DINSTALL_MYSQLSHAREDIR=share/mariadb \
                	-DINSTALL_LAYOUT=RPM \
                	-DINSTALL_PLUGINDIR=lib/mariadb/plugin \
                	-DINSTALL_MYSQLDATADIR=/srv/mysql \
                	-DINSTALL_UNIX_ADDRDIR=/var/run/mysql.sock
                        $(CMAKE_HOST_OPTIONS) \
	..
endef

define Host/Compile
	cd $(HOST_BUILD_DIR)/build; make import_executables
endef

define Host/Install
	mkdir -p $(STAGING_DIR_HOST); \
	cd $(HOST_BUILD_DIR); $(CP) build $(STAGING_DIR_HOST)/mariadb-import_executables
endef

define Build/InstallDev
	$(INSTALL_DIR) $(2)/bin $(1)/usr/bin $(1)/usr/include $(1)/usr/lib
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/mysql_config $(1)/usr/bin/
	ln -sf $(STAGING_DIR)/usr/bin/mysql_config $(2)/bin/
	$(CP) $(PKG_INSTALL_DIR)/usr/include/mysql $(1)/usr/include/
	# NOTE: needed for MySQL-Python
	$(CP) $(PKG_BUILD_DIR)/include/mysqld_error.h $(1)/usr/include/mysql/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/mariadb $(1)/usr/lib/
	rm -f $(1)/usr/lib/*.la
endef

define Package/libmariadbclient/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libmariadb*.so.* $(1)/usr/lib/
endef

define Package/mariadb-server/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/* $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/sbin/mysqld $(1)/usr/sbin/
	$(RM) $(1)/usr/bin/resolveip $(1)/usr/bin/mysqltest
	$(INSTALL_DIR) $(1)/usr/lib/mariadb/plugin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/lib/mariadb/plugin/* $(1)/usr/lib/mariadb/plugin
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) files/mysqld.init $(1)/etc/init.d/mysqld
	$(INSTALL_CONF) $(PKG_INSTALL_DIR)/usr/share/mysql/my-small.cnf $(1)/etc/my.cnf
	$(INSTALL_DIR) $(1)/usr/share/mariadb
	$(INSTALL_DIR) $(1)/usr/share/mariadb/english
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/share/mariadb/english/errmsg.sys $(1)/usr/share/mariadb/english
	$(CP) $(PKG_INSTALL_DIR)/usr/share/mariadb/charsets $(1)/usr/share/mariadb
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/share/mariadb/*.sql $(1)/usr/share/mariadb/
endef

define Package/mariadb-server/conffiles
/etc/my.cnf
endef

$(eval $(call BuildPackage,mariadb-server))
$(eval $(call BuildPackage,libmariadbclient))
$(eval $(call HostBuild))
