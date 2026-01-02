# Writing Custom Modules for OpenWrt

This document describes how to add a custom kernel module (kmod) package to
the OpenWrt build system. The same structure applies to user-space packages,
but kmods use the kernel build system and install to `/lib/modules/`.

## 1. Create the Package Skeleton

Pick a package name (example: `kmod-foo`) and create a new directory:

```
package/kmod-foo/
  Makefile
  src/
    foo.c
    Makefile
  files/
    test_foo.sh
```

Notes:
- Keep source under `src/` for out-of-tree modules.
- Use `files/` for scripts or tests that should be staged into the rootfs.

## 2. Kconfig Entry (optional but common)

If the package should be selectable in `menuconfig`, add it in the package
Makefile (see below). You do not need a separate `Config.in` unless you are
adding new global Kconfig options.

## 3. Package Makefile (Kernel Module)

Create `package/kmod-foo/Makefile`:

```make
include $(TOPDIR)/rules.mk

PKG_NAME:=kmod-foo
PKG_RELEASE:=1

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/foo
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Other modules
  TITLE:=Foo kernel module
  FILES:=$(PKG_BUILD_DIR)/foo.ko
  AUTOLOAD:=$(call AutoLoad,50,foo)
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" modules
endef

$(eval $(call KernelPackage,foo))
```

Key points:
- Use `KernelPackage` for kmods, not `BuildPackage`.
- `FILES` must point at the built `.ko`.
- `AUTOLOAD` controls module loading order; omit if not needed.

## 4. Kernel Module Source

`package/kmod-foo/src/Makefile`:

```make
obj-m += foo.o
```

`package/kmod-foo/src/foo.c` (minimal example):

```c
#include <linux/module.h>

static int __init foo_init(void)
{
	return 0;
}

static void __exit foo_exit(void)
{
}

module_init(foo_init);
module_exit(foo_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Example OpenWrt kernel module");
```

## 5. Build and Test

From the OpenWrt root:

```
make package/kmod-foo/compile V=sc
make package/kmod-foo/install
```

Artifacts:
- `.ko` will be staged under `build_dir/target-*/` and packaged into an `.ipk`.
- Final images land in `bin/targets/<target>/<subtarget>/`.

Optional tests:
- Place a simple test in `package/kmod-foo/files/test_foo.sh`.
- Keep it POSIX shell and name it `test_<feature>.sh` for discovery.

## 6. Common Pitfalls

- Use `TARGET_*` and `KERNEL_MAKE`; do not call host `gcc`.
- Do not commit `.config`; use `./scripts/diffconfig.sh`.
- Keep Makefile recipe lines tab-indented.

## 7. User-Space "Module" (Regular Package)

For user-space code (daemons, tools, libraries), use `BuildPackage` and
install into `/usr/` (or another rootfs path).

Example `package/foo/Makefile`:

```make
include $(TOPDIR)/rules.mk

PKG_NAME:=foo
PKG_VERSION:=1.0
PKG_RELEASE:=1

PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/foo
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Foo user-space tool
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(TARGET_CC) $(TARGET_CFLAGS) -o $(PKG_BUILD_DIR)/foo $(PKG_BUILD_DIR)/foo.c
endef

define Package/foo/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/foo $(1)/usr/bin/
endef

$(eval $(call BuildPackage,foo))
```

Notes:
- Prefer `cmake.mk`, `autotools.mk`, or `meson.mk` when upstream supports them.
- Use `TARGET_*` variables to respect cross-compilation.

## 8. Example: External Source Tarball

If your module source is hosted externally, use `PKG_SOURCE` and
`PKG_SOURCE_URL` and let OpenWrt fetch and unpack it.

Example `package/kmod-bar/Makefile`:

```make
include $(TOPDIR)/rules.mk

PKG_NAME:=kmod-bar
PKG_VERSION:=2.3.4
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://example.com/releases/
PKG_HASH:=skip

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/bar
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Other modules
  TITLE:=Bar kernel module
  FILES:=$(PKG_BUILD_DIR)/bar.ko
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" modules
endef

$(eval $(call KernelPackage,bar))
```

Notes:
- Replace `PKG_HASH:=skip` with the real SHA256 for reproducible builds.
- If the tarball has its own subdirectory layout, set `PKG_BUILD_DIR` to match.

## 9. Kernel-Tree Modules (In-Tree)

If the module already exists in the Linux kernel tree, do not copy sources.
Instead, enable it via `KCONFIG` and package the `.ko` from the kernel build.

Example `package/kmod-baz/Makefile`:

```make
include $(TOPDIR)/rules.mk

PKG_NAME:=kmod-baz
PKG_RELEASE:=1

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/baz
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Other modules
  TITLE:=Baz in-tree module
  KCONFIG:=CONFIG_BAZ
  FILES:=$(LINUX_DIR)/drivers/baz/baz.ko
  AUTOLOAD:=$(call AutoLoad,60,baz)
endef

$(eval $(call KernelPackage,baz))
```

Notes:
- Set `KCONFIG` to the kernel option that builds the module.
- Point `FILES` at the built `.ko` within the kernel tree.

## 10. Duplicating an Existing Kmod Package

Copying a kmod package directory is only the starting point. You must update
names and outputs to avoid collisions with the original package.

Checklist (example: `gpio-button-hotplug` → `gpio-button-hotplug-ng`):
- Copy directory: `package/kernel/gpio-button-hotplug` →
  `package/kernel/gpio-button-hotplug-ng`.
- Update `PKG_NAME`, `KernelPackage/<name>`, `TITLE`, and description.
- Update `FILES` to match the new `.ko` name.
- Update `AUTOLOAD` to reference the new module name.
- In `src/Makefile`, rename `obj-m` target to the new module object.
- Rename the `.c` file to match the new module name and update
  `MODULE_*` strings inside the source.
- Build with `make package/gpio-button-hotplug-ng/compile V=sc`.
