# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenWrt is a Linux distribution for embedded devices (routers, IoT devices, etc.). This is the core build system that cross-compiles the toolchain, kernel, and packages to generate firmware images for specific hardware targets.

The build system is make-based and highly modular, using:
- **Kconfig** for configuration (like Linux kernel)
- **Feed system** for external package repositories
- **Target/subtarget architecture** for hardware-specific builds
- **Staging directories** for cross-compilation artifacts

## Core Architecture

### Directory Structure

- `target/linux/` - Hardware platform definitions (SoC families like realtek, ath79, ipq40xx, etc.)
  - Each target has a `Makefile` defining ARCH, BOARD, FEATURES, SUBTARGETS, and KERNEL_PATCHVER
  - Contains device-specific DTS files, kernel patches, and image generation recipes
- `package/` - Package definitions (base system, boot loaders, kernel modules, utilities)
  - Each package has a `Makefile` with PKG_* variables and Build/* functions
  - Organized by category: base-files, boot/, kernel/, firmware/, devel/, network/, utils/
- `include/` - Build system core (.mk files)
  - `package.mk` - Package build framework
  - `kernel.mk`, `kernel-build.mk` - Kernel compilation
  - `image.mk`, `image-commands.mk` - Firmware image generation
  - `download.mk` - Source fetching
  - `autotools.mk`, `cmake.mk`, `meson.mk` - Build system integration
- `scripts/` - Build automation scripts (mostly shell and Perl)
  - `feeds` - Feed management
  - `diffconfig.sh` - Generate minimal config diffs
  - `checkpatch.pl` - Code style checker (from Linux kernel)
  - Image manipulation scripts (Python)
- `toolchain/` - Cross-compilation toolchain (gcc, binutils, libc)
- `tools/` - Host utilities needed during build
- `rules.mk` - Core build system variables and functions
- `feeds.conf.default` - Default external package feed sources (can be overridden by `feeds.conf`)

### Build Flow

1. **Configuration**: `make menuconfig` generates `.config` (Kconfig)
2. **Tools**: Build host tools in `tools/` → `staging_dir/host/`
3. **Toolchain**: Build cross-compiler → `staging_dir/toolchain-*/`
4. **Kernel**: Compile kernel for target → `build_dir/target-*/linux-*/`
5. **Packages**: Cross-compile packages → `build_dir/target-*/*/`
6. **Rootfs**: Assemble root filesystem with selected packages
7. **Image**: Generate flashable firmware → `bin/targets/<target>/<subtarget>/`

Build artifacts:
- `staging_dir/` - Cross-compilation staging area, host tools
- `build_dir/` - Temporary build directories
- `dl/` - Downloaded source archives
- `bin/` - Final firmware images and packages
- `tmp/` - Temporary files

### Package Makefile Structure

Standard OpenWrt package `Makefile`:

```make
include $(TOPDIR)/rules.mk

PKG_NAME:=example
PKG_VERSION:=1.0
PKG_RELEASE:=1
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://example.com/
PKG_BUILD_DEPENDS:=...  # Build-time dependencies

include $(INCLUDE_DIR)/package.mk

define Package/example
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Example package
  DEPENDS:=+libc +libpthread  # Runtime dependencies
endef

define Build/Configure
  # Configure step (often uses autotools.mk or cmake.mk)
endef

define Build/Compile
  # Compilation (cross-compile with TARGET_CC, TARGET_CFLAGS)
endef

define Package/example/install
  $(INSTALL_DIR) $(1)/usr/bin
  $(INSTALL_BIN) $(PKG_BUILD_DIR)/example $(1)/usr/bin/
endef

$(eval $(call BuildPackage,example))
```

Variables:
- `PKG_*` - Package metadata
- `TARGET_*` - Cross-compilation environment (TARGET_CC, TARGET_CFLAGS, TARGET_LDFLAGS)
- `STAGING_DIR` - Sysroot for finding libraries
- `$(1)` in install functions - Package staging root

## Common Development Commands

### Initial Setup
```bash
# Update package feeds from remote repositories
./scripts/feeds update -a

# Install feed packages (creates symlinks in package/feeds/)
./scripts/feeds install -a

# Configure target and packages (ncurses menu)
make menuconfig

# Alternative: Load a minimal config diff
# cp configs/mydevice.config .config
# make defconfig
```

### Building
```bash
# Full build (single-core or multi-core)
make
# Or with parallel jobs for faster builds
make -j$(nproc)

# Build with verbose output
make V=s

# Build single package (faster iteration)
make package/<pkgname>/compile V=sc

# Clean and rebuild package
make package/<pkgname>/clean
make package/<pkgname>/compile V=sc

# Install package to staging
make package/<pkgname>/install

# Build only kernel
make target/linux/compile

# Build only toolchain
make toolchain/compile
```

### Configuration
```bash
# Generate minimal config diff (for sharing/version control)
./scripts/diffconfig.sh > my.config

# Apply config and expand to full .config
cp my.config .config
make defconfig

# Update kernel config for a target
make kernel_menuconfig
```

### Cleaning
```bash
# Clean packages and images (keep toolchain and downloads)
make clean

# Clean everything except dl/ (deep clean)
make dirclean

# Clean only target packages (keep toolchain)
make targetclean

# Clean download cache
rm -rf dl/
```

### Development Tools
```bash
# Update kernel patches with quilt
make target/linux/refresh

# Check code style for patches
./scripts/checkpatch.pl --strict <patchfile>

# Validate package dependencies
make package/symlinks
make prereq

# List available feeds
./scripts/feeds list

# Search for a package in feeds
./scripts/feeds search <pattern>

# Install specific package from feed
./scripts/feeds install <pkgname>
```

## Testing Builds

### Building for a Specific Target
1. `make menuconfig`
   - Select "Target System" (e.g., Atheros AR7xxx/AR9xxx)
   - Select "Subtarget" (e.g., Generic devices with NAND flash)
   - Select "Target Profile" (e.g., Netgear WNDR4300)
   - Select packages in Categories
2. `make -j$(nproc) V=s` - Build with verbose output
3. Test image in `bin/targets/<target>/<subtarget>/` on hardware or QEMU

### Testing Individual Packages
```bash
# Build package with verbose output and compiler commands
make package/<pkgname>/compile V=sc

# Check installed files
ls -la build_dir/target-*/*/ipkg-*/

# Inspect package contents
tar -tvf bin/packages/*/base/<pkgname>*.ipk

# Test on running OpenWrt device (via SCP + opkg)
scp bin/packages/*/base/<pkgname>*.ipk root@router:/tmp/
ssh root@router "opkg install /tmp/<pkgname>*.ipk"
```

## Code Style & Conventions

### Makefiles
- Tab indentation (GNU Make standard)
- UPPERCASE variable names (PKG_VERSION, DEPENDS)
- Order: source variables → build variables → package definitions → build functions → eval
- Use `$(INSTALL_DIR)`, `$(INSTALL_BIN)`, `$(INSTALL_DATA)` macros for install functions

### Kconfig (Config.in)
- Tabs for keywords (config, bool, depends)
- Two spaces for help text indentation

### C Code Patches
- Must pass `./scripts/checkpatch.pl --strict`
- Follow Linux kernel coding style
- Patches stored in `target/linux/<target>/patches-*/` or `package/<pkg>/patches/`

### Shell Scripts
- POSIX-compliant (#!/bin/sh)
- snake_case function names
- Use `set -e` for error handling (when appropriate)
- Place in package's `files/` subdirectory

## Commit Conventions

Follow existing git history style:

```
<subsystem>: <short description>

<Detailed explanation of the change, motivation, and impact>

Signed-off-by: Your Name <email>
Fixes: #<issue>
```

Examples:
- `realtek: dsa: clarify lag state variable meaning`
- `kernel: sound: add support for MIDI 2.0 and UMP`
- `ipq40xx: fix unit-address of Netgear LBR20 ubi partition`

Subsystems: target names (e.g., `ath79`, `ipq40xx`), package names, `kernel`, `build`, `scripts`, `toolchain`

## Important Notes

### Configuration Management
- **Never commit `.config`** - it contains full expanded configuration
- **Always use diffconfig** for sharing: `./scripts/diffconfig.sh > configs/feature.config`
- Store minimal configs in a `configs/` directory (not tracked by default)

### Cross-Compilation
- Always use `TARGET_CC`, not `$(CC)` in package Makefiles
- Use `TARGET_CFLAGS`, `TARGET_LDFLAGS` for proper cross-compilation
- Link against libraries in `$(STAGING_DIR)`, not host system

### Kernel Modules
- Kernel module packages use `KernelPackage` instead of `BuildPackage`
- Module .ko files installed to `/lib/modules/$(LINUX_VERSION)/`

### Image Generation
- Image recipes in `target/linux/<target>/image/*.mk`
- Uses functions from `include/image-commands.mk` (e.g., append-kernel, append-rootfs)
- Device-specific recipes use `Device/*` variables

### Package Dependencies
- Runtime dependencies: `DEPENDS:=+package +@feature`
- Build dependencies: `PKG_BUILD_DEPENDS:=package/host`
- `+` prefix = runtime dependency
- `/host` suffix = host tool dependency

### Feeds System
- External repos defined in `feeds.conf.default` (or custom `feeds.conf`)
- After modifying feeds configuration: `./scripts/feeds update -a && ./scripts/feeds install -a`
- Feed packages appear in `package/feeds/<feedname>/`

## References

- OpenWrt Developer Guide: https://openwrt.org/docs/guide-developer/start
- Build System: https://openwrt.org/docs/guide-developer/build-system/start
- Package Makefiles: https://openwrt.org/docs/guide-developer/packages
- Kernel: https://openwrt.org/docs/guide-developer/kernel
