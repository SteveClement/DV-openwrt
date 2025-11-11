# OpenWrt Build System

## Project Overview

This directory contains the source code for the OpenWrt build system. OpenWrt is a highly extensible GNU/Linux distribution for embedded devices. The build system is based on `make` and allows for the configuration and compilation of a custom firmware image for a wide range of target devices.

The project is organized into several key directories:

*   `package/`: Contains the definitions for all the software packages that can be included in the firmware.
*   `target/`: Contains the target-specific configurations for different hardware platforms.
*   `toolchain/`: Contains the tools and libraries needed to build the cross-compilation toolchain.
*   `scripts/`: A collection of scripts, mostly in shell and Perl, that automate the build process.
*   `include/`: Contains common makefiles and headers used throughout the build system.
*   `feeds.conf.default`: Defines the default package feeds, which are external repositories of software packages.

## Building and Running

The build process is managed by `make` and is highly configurable. The following steps provide a general overview of the build process:

1.  **Update Feeds:**
    ```bash
    ./scripts/feeds update -a
    ```
    This command downloads the latest package definitions from the repositories listed in `feeds.conf.default`.

2.  **Install Feeds:**
    ```bash
    ./scripts/feeds install -a
    ```
    This command creates symlinks to the package definitions in the `package/feeds` directory, making them available to the build system.

3.  **Configure:**
    ```bash
    make menuconfig
    ```
    This command opens a menu-driven interface that allows you to select the target device, packages to include, and other configuration options.

4.  **Build:**
    ```bash
    make
    ```
    This command starts the build process. The build system will download the source code for all the selected packages, build the cross-compilation toolchain, and compile the kernel and all the packages for the target device.

The final firmware images will be located in the `bin/targets/<target>/<subtarget>/` directory.

## Development Conventions

*   **Makefiles:** The build system is heavily based on `make`. Each package has its own `Makefile` that defines how it should be compiled and installed.
*   **Kconfig:** The configuration system is based on `Kconfig`, the same system used by the Linux kernel. This allows for a flexible and powerful configuration system.
*   **Feeds:** The package management system is based on the concept of "feeds". Feeds are external repositories of packages that can be easily added to the build system.
*   **Patching:** The build system has a mechanism for applying patches to the source code of packages. This is useful for fixing bugs or adding new features.
*   **Scripts:** A large number of scripts are used to automate the build process. These scripts are mostly written in shell and Perl.
