#
# Copyright (C) 2002-2003 Erik Andersen <andersen@uclibc.org>
# Copyright (C) 2004 Manuel Novoa III <mjn3@uclibc.org>
# Copyright (C) 2005-2006 Felix Fietkau <nbd@nbd.name>
# Copyright (C) 2006-2014 OpenWrt.org
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA

include $(TOPDIR)/rules.mk

PKG_NAME:=gcc
PKG_VERSION:=13.2.0
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=@GNU/gcc/gcc-$(PKG_VERSION)
PKG_HASH:=e275e76442a6067341a27f04c5c6b83d8613144004c0413528863dc6b5c743da

PKG_INSTALL:=1
PKG_FIXUP:=libtool
PKG_BUILD_PARALLEL:=1

PKG_CPE_ID:=cpe:/a:gnu:gcc

PATCH_DIR=./patches/13.2.0

include $(INCLUDE_DIR)/package.mk

define Package/gcc
  SECTION:=devel
  CATEGORY:=Development
  TITLE:=gcc
  MAINTAINER:=W. Michael Petullo <mike@flyn.org>
  DEPENDS:= +binutils +libstdcpp +libzstd @!arc
  MENU:=1
endef

define Package/gcc-extras
  SECTION:=devel
  CATEGORY:=Development
  TITLE:=gcc-extras
  MAINTAINER:=Oskari Rauta <oskari.rauta@gmail.com>
  DEPENDS:= gcc +libstdcpp @!arc
endef

define Package/libc-static
  SECTION:=devel
  CATEGORY:=Development
  SUBMENU:=Static libraries
  TITLE:=libc static library
  MAINTAINER:=Oskari Rauta <oskari.rauta@gmail.com>
endef

define Package/libpthread-static
  SECTION:=devel
  CATEGORY:=Development
  SUBMENU:=Static libraries
  TITLE:=libpthread static library
  MAINTAINER:=Oskari Rauta <oskari.rauta@gmail.com>
endef

define Package/libstdcpp-static
  SECTION:=devel
  CATEGORY:=Development
  SUBMENU:=Static libraries
  TITLE:=libstdcpp static library
  MAINTAINER:=Oskari Rauta <oskari.rauta@gmail.com>
endef

define Package/gcc/description
  gcc and g++ compilers
endef

define Package/gcc-extras/description
  sanitizers and other extras are as static and dynamic included in this package
endef

define Package/libc-static/description
  This package contains static libc
endef

define Package/libpthread-static/description
  This package contains static libpthread
endef

define Package/libstdcpp-static/description
  This package contains static libstdc++
endef

TARGET_LANGUAGES:="c,c++"
BUGURL=https://dev.openwrt.org/
PKGVERSION=OpenWrt GCC $(PKG_VERSION)
TARGET_CPPFLAGS += -D_GLIBCXX_INCLUDE_NEXT_C_HEADERS

# not using sstrip here as this messes up the .so's somehow
STRIP:=$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)strip
RSTRIP:= \
	NM="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)nm" \
	STRIP="$(STRIP)" \
	STRIP_KMOD="$(STRIP) --strip-debug" \
	$(SCRIPT_DIR)/rstrip.sh

ifneq ($(CONFIG_SOFT_FLOAT),y)
	ifeq ($(CONFIG_arm),y)
		ARM_FLOAT_OPTION:= --with-float=hard
	endif
endif

GMPSRC=gmp-6.1.0

define Download/gmp
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(GMPSRC).tar.bz2
  HASH:=498449a994efeba527885c10405993427995d3f86b8768d8cdf8d9dd7c6b73e8
endef
$(eval $(call Download,gmp))

MPCSRC=mpc-1.0.3

define Download/mpc
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(MPCSRC).tar.gz
  HASH:=617decc6ea09889fb08ede330917a00b16809b8db88c29c31bfbb49cbf88ecc3
endef
$(eval $(call Download,mpc))

MPFRSRC=mpfr-3.1.4

define Download/mpfr
  URL:=ftp://gcc.gnu.org/pub/gcc/infrastructure/
  FILE:=$(MPFRSRC).tar.bz2
  HASH:=d3103a80cdad2407ed581f3618c4bed04e0c92d1cf771a65ead662cc397f7775
endef
$(eval $(call Download,mpfr))

define Build/Prepare
	$(PKG_UNPACK)
#	we have to download and unpack additional stuff before patching
	tar -C $(PKG_BUILD_DIR) -xvjf $(DL_DIR)/$(GMPSRC).tar.bz2
	ln -sf $(PKG_BUILD_DIR)/$(GMPSRC) $(PKG_BUILD_DIR)/gmp
	tar -C $(PKG_BUILD_DIR) -xvzf $(DL_DIR)/$(MPCSRC).tar.gz
	ln -sf $(PKG_BUILD_DIR)/$(MPCSRC) $(PKG_BUILD_DIR)/mpc
	tar -C $(PKG_BUILD_DIR) -xvjf $(DL_DIR)/$(MPFRSRC).tar.bz2
	ln -sf $(PKG_BUILD_DIR)/$(MPFRSRC) $(PKG_BUILD_DIR)/mpfr
	$(Build/Patch)
#	poor man's fix for `none-openwrt-linux' not recognized when building with musl
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/mpfr/
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/gmp/
	chmod u+w $(PKG_BUILD_DIR)/mpc/config.sub
	cp $(PKG_BUILD_DIR)/config.sub $(PKG_BUILD_DIR)/mpc/
endef

CONFIGURE_ARGS += CXXFLAGS_FOR_TARGET="-g -O2 -D_GLIBCXX_INCLUDE_NEXT_C_HEADERS"

define Build/Configure
	(cd $(PKG_BUILD_DIR); rm -f config.cache; \
		SHELL="$(BASH)" \
		$(TARGET_CONFIGURE_OPTS) \
		$(PKG_BUILD_DIR)/configure \
			$(CONFIGURE_ARGS) \
			--build=$(GNU_HOST_NAME) \
			--host=$(REAL_GNU_TARGET_NAME) \
			--target=$(REAL_GNU_TARGET_NAME) \
			--enable-languages=$(TARGET_LANGUAGES) \
			--with-bugurl=$(BUGURL) \
			--with-pkgversion="$(PKGVERSION)" \
			--enable-shared \
			$(if $(CONFIG_LIBC_USE_GLIBC),--enable,--disable)-__cxa_atexit \
			--with-default-libstdcxx-abi=gcc4-compatible \
			--enable-target-optspace \
			--with-gnu-ld \
			--disable-nls \
			--enable-__cxa_atexit \
			--enable-lto \
			--disable-libsanitizer \
			--disable-libvtv \
			--disable-libcilkrts \
			--disable-libmudflap \
			--disable-libmpx \
			--disable-multilib \
			--disable-libgomp \
			--disable-libquadmath \
			--disable-libssp \
			--disable-decimal-float \
			--disable-libstdcxx-pch \
			--with-host-libstdcxx=-lstdc++ \
			--prefix=/usr \
			--libexecdir=/usr/lib \
			--with-local-prefix=/usr \
			--with-stage1-ldflags=-lstdc++ \
			$(ARM_FLOAT_OPTION) \
			$(SOFT_FLOAT_CONFIG_OPTION) \
			$(call qstrip,$(CONFIG_EXTRA_GCC_CONFIG_OPTIONS)) \
	);
endef

define Build/Compile
	export SHELL="$(BASH)"; $(MAKE_VARS) $(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
			DESTDIR="$(PKG_INSTALL_DIR)" $(MAKE_ARGS) all
	export SHELL="$(BASH)"; $(MAKE_VARS) $(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
			DESTDIR="$(PKG_INSTALL_DIR)" $(MAKE_ARGS) install
endef

ENVCFLAGS:="$(TARGET_OPTIMIZATION) $(EXTRA_OPTIMIZATION)
ifeq ($(CONFIG_SOFT_FLOAT),y)
    ifeq ($(CONFIG_arm),y)
	ENVCFLAGS+= -mfloat-abi=soft
    else
	ENVCFLAGS+= -msoft-float
    endif
endif
ENVCFLAGS+="

ENVLDFLAGS:="-Wl,-rpath=/usr/lib -Wl,--dynamic-linker=/usr/lib/$(DYNLINKER) -L/usr/lib, -lstdc++"

define Package/gcc/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/usr/lib $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -ar $(PKG_INSTALL_DIR)/usr/include $(1)/usr
	cp -a $(PKG_INSTALL_DIR)/usr/bin/{$(REAL_GNU_TARGET_NAME)-{g++,gcc},cpp,gcov} $(1)/usr/bin
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/c++
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/g++
	ln -s $(REAL_GNU_TARGET_NAME)-g++ $(1)/usr/bin/$(REAL_GNU_TARGET_NAME)-c++
	ln -s $(REAL_GNU_TARGET_NAME)-gcc $(1)/usr/bin/gcc
	ln -s $(REAL_GNU_TARGET_NAME)-gcc $(1)/usr/bin/cc
	ln -s $(REAL_GNU_TARGET_NAME)-gcc $(1)/usr/bin/$(REAL_GNU_TARGET_NAME)-gcc-$(PKG_VERSION)
	cp -ar $(PKG_INSTALL_DIR)/usr/lib/gcc $(1)/usr/lib
	cp -ar $(TOOLCHAIN_DIR)/include $(1)/usr
	cp -a $(TOOLCHAIN_DIR)/lib/*.{o,so*} $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/*nonshared*.a  $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/libm.a  $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)

	rm -f $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/libgo*
	rm -f $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)/libcc1*
	echo '#!/bin/sh' > $(1)/usr/bin/gcc_env.sh
	echo 'export LDFLAGS=$(ENVLDFLAGS)' >> $(1)/usr/bin/gcc_env.sh
	echo 'export CFLAGS=$(ENVCFLAGS)' >> $(1)/usr/bin/gcc_env.sh
	chmod +x $(1)/usr/bin/gcc_env.sh
endef

define Package/gcc-extras/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/usr/lib
	cp -a $(PKG_INSTALL_DIR)/usr/bin/{gcov-dump,gcov-tool,lto-dump} $(1)/usr/bin
	cp -a $(PKG_INSTALL_DIR)/usr/bin/$(REAL_GNU_TARGET_NAME)-gcc-ar $(1)/usr/bin
	cp -a $(PKG_INSTALL_DIR)/usr/bin/$(REAL_GNU_TARGET_NAME)-gcc-nm $(1)/usr/bin
	ln -s $(REAL_GNU_TARGET_NAME)-gcc-ar $(1)/usr/bin/gcc-ar
	ln -s $(REAL_GNU_TARGET_NAME)-gcc-nm $(1)/usr/bin/$(REAL_GNU_TARGET_NAME)-ranlib
	ln -s $(REAL_GNU_TARGET_NAME)-gcc-nm $(1)/usr/bin/gcc-nm
	ln -s $(REAL_GNU_TARGET_NAME)-gcc-nm $(1)/usr/bin/gcc-ranlib

	# Sanitizers and other extras..
	#cp -a $(PKG_INSTALL_DIR)/usr/lib/{libasan.a,libasan.la,libasan.so.8.0.0,libasan_preinit.o} $(1)/usr/lib
	#ln -s libasan.so.8.0.0 $(1)/usr/lib/libasan.so
	#ln -s libasan.so.8.0.0 $(1)/usr/lib/libasan.so.8
	#cp -a $(PKG_INSTALL_DIR)/usr/lib/{libhwasan.a,libhwasan.la,libhwasan.so.0.0.0,libhwasan_preinit.o} $(1)/usr/lib
	#ln -s libhwasan.so.0.0.0 $(1)/usr/lib/libhwasan.so
	#ln -s libhwasan.so.0.0.0 $(1)/usr/lib/libhwasan.so.0
	cp -a $(PKG_INSTALL_DIR)/usr/lib/{libitm.a,libitm.la,libitm.so.1.0.0,libitm.spec} $(1)/usr/lib
	ln -s libitm.so.1.0.0 $(1)/usr/lib/libitm.so.1
	ln -s libitm.so.1.0.0 $(1)/usr/lib/libitm.so
	#cp -a $(PKG_INSTALL_DIR)/usr/lib/{liblsan.a,liblsan.la,liblsan.so.0.0.0,liblsan_preinit.o,libsanitizer.spec} $(1)/usr/lib
	#ln -s liblsan.so.0.0.0 $(1)/usr/lib/liblsan.so.0
	#ln -s liblsan.so.0.0.0 $(1)/usr/lib/liblsan.so
	#cp -a $(PKG_INSTALL_DIR)/usr/lib/{libtsan.a,libtsan.la,libtsan.so.2.0.0,libtsan_preinit.o} $(1)/usr/lib
	#ln -s libtsan.so.2.0.0 $(1)/usr/lib/libtsan.so.2
	#ln -s libtsan.so.2.0.0 $(1)/usr/lib/libtsan.so
	#cp -a $(PKG_INSTALL_DIR)/usr/lib/{libubsan.a,libubsan.la,libubsan.so.1.0.0} $(1)/usr/lib
	#ln -s libubsan.so.1.0.0 $(1)/usr/lib/libubsan.so.1
	#ln -s libubsan.so.1.0.0 $(1)/usr/lib/libubsan.so
	cp -a $(PKG_INSTALL_DIR)/usr/lib/{libstdc++exp.a,libstdc++exp.la} $(1)/usr/lib
	cp -a $(PKG_INSTALL_DIR)/usr/lib/{libstdc++fs.a,libstdc++fs.la} $(1)/usr/lib
	cp -a $(PKG_INSTALL_DIR)/usr/lib/{libsupc++.a,libsupc++.la} $(1)/usr/lib
endef

define Package/libc-static/install
	$(INSTALL_DIR) $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/libc.a $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
endef

define Package/libpthread-static/install
	$(INSTALL_DIR) $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/libpthread.a $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
endef

define Package/libstdcpp-static/install
	$(INSTALL_DIR) $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
	cp -a $(TOOLCHAIN_DIR)/lib/libstdc++.a $(1)/usr/lib/$(PKG_NAME)/$(REAL_GNU_TARGET_NAME)/$(PKG_VERSION)
endef

$(eval $(call BuildPackage,gcc))
$(eval $(call BuildPackage,gcc-extras))
$(eval $(call BuildPackage,libc-static))
$(eval $(call BuildPackage,libpthread-static))
$(eval $(call BuildPackage,libstdcpp-static))
