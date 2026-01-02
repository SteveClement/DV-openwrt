# Integrating dsniff into OpenWrt

This document captures the required pieces to add `dsniff` and its missing
dependencies to this tree. It includes draft package Makefiles and build
notes for common issues.

## Upstream Sources

- dsniff: `https://www.monkey.org/~dugsong/dsniff/` (latest `2.3`)
- libnet: `https://github.com/libnet/libnet` (latest `1.3`)
- libnids: `https://github.com/MITRECND/libnids` (tag `1.26`)

## Dependency Mapping

dsniff depends on:
- Berkeley DB with 1.85 compatibility: OpenWrt `libdb47` (from `openwrt/packages`)
- OpenSSL: `libopenssl` (already in this tree)
- libpcap: `libpcap` (already in this tree)
- libnet: **missing** (add package)
- libnids: **missing** (add package)

## Package Files Added

The draft packages below live at:
- `package/libs/libnet/Makefile`
- `package/libs/libnids/Makefile`
- `package/network/utils/dsniff/Makefile`

## Draft Package Makefiles

These are included in-tree and ready for iteration. Update versions and hashes
if you decide to track different releases.

## Build Notes and Known Issues

- dsniff's `configure` checks for `db_185.h` and `libdb.a`. OpenWrt's `libdb47`
  is built with `--enable-compat185`, but may not ship static `libdb.a` by
  default. If `configure` fails, patch it to accept shared libs or inject
  `DBLIB="-ldb"` via `CONFIGURE_VARS`.
- `libnet-config` is used by dsniff's `configure` to set CFLAGS. Ensure
  `libnet` installs it into staging and that `PATH` includes `$(STAGING_DIR)/usr/bin`.
- `webspy` uses X11 and is skipped by `--without-x`. The OpenWrt package
  disables X to avoid X11 deps.
- `tcphijack` is optional and usually not built on OpenWrt; the package install
  loop ignores missing binaries.

## Suggested Build Commands

```
make package/libnet/compile V=sc
make package/libnids/compile V=sc
make package/dsniff/compile V=sc
```

## Licensing

- dsniff: BSD-3-Clause (from upstream `LICENSE`)
- libnet: BSD-2-Clause (from repo metadata)
- libnids: GPL-2.0-only (from upstream `COPYING`)
