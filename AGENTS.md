# Repository Guidelines

## Project Structure & Module Organization
The top-level `Makefile` orchestrates the full toolchain build, while `target/` holds SoC-specific kernels, DTS files, and image recipes. Place new packages under `package/`, keeping helper scripts in `scripts/` and reusable build helpers in `include/`. Host-side utilities reside in `tools/` and `toolchain/`, and device configuration defaults belong in `config/` and the relevant `Config.in` fragment. Images and SDK artifacts are emitted to `bin/targets/<target>/<subtarget>/`.

## Build, Test, and Development Commands
- `./scripts/feeds update -a` – sync every feed listed in `feeds.conf.default`.
- `./scripts/feeds install -a` – stage feed packages into `package/feeds/`.
- `make menuconfig` – configure targets, toolchain options, and package sets via ncurses UI.
- `make -j$(nproc)` – perform the full toolchain, kernel, and rootfs build; artifacts land in `bin/`.
- `make package/<pkgname>/{clean,compile} V=sc` – rebuild a single package with verbose logs for quicker iteration.

## Coding Style & Naming Conventions
Package `Makefile`s follow the Linux kernel style: tab-indented recipes, uppercase variable names (`PKG_RELEASE`, `DEPENDS`), and sections ordered source ➝ build ➝ install. Kconfig fragments in `Config.in` use tabs for keywords and two spaces for help text. C patches should pass `scripts/checkpatch.pl --strict`, and shell helpers must be POSIX-compliant, using `snake_case` function names and `set -euo pipefail` where practical.

## Testing Guidelines
Always exercise incremental builds before pushing: `make package/<pkgname>/compile V=sc` validates cross-compilation, and `make package/<pkgname>/install` inspects the staged root. Boot-test the resulting image on real hardware or QEMU using the files under `bin/targets/...`. Store functional tests beside the package (e.g., `package/foo/files/tests/test_wireless.sh`) and name them `test_<feature>.sh` so farm runners can auto-discover them. Capture throughput, boot logs, or regression diffs and attach them to your review notes.

## Commit & Pull Request Guidelines
Match the existing history (`subsystem: short synopsis`, e.g., `realtek: dsa: clarify lag state`). Keep the subject under 72 characters, explain motivation and user impact in the body, and include `Signed-off-by` plus any `Fixes #<issue>` tags. Pull requests should describe the target device, configuration snippet (`diffconfig`), and test evidence (log excerpt or checksum). Group logically-related commits only; rebase onto the latest `master` before opening the PR.

## Security & Configuration Tips
Do not commit personal `.config` files; instead, share minimal diffs created with `./scripts/diffconfig.sh > configs/<feature>.diffconfig`. Scrub device-specific keys or certificates from `files/` overlays, and prefer referencing secrets via UCI defaults or documentation links.
