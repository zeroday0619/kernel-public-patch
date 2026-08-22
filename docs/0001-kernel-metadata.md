# Kernel Identity and Build Metadata Patch

## Overview

[`0001-feat-kernel-expose-zeroday0619-kernel-metadata.patch`](../patches/0001-kernel-metadata/0001-feat-kernel-expose-zeroday0619-kernel-metadata.patch)
adds structured identity and build metadata for the zeroday0619 kernel
distribution. It exposes immutable values through sysfs and replaces the normal
local-version composition with a Kconfig-driven release string.

This is a kernel identification patch. It does not add scheduler, compiler, or
other performance optimizations.

## Compatibility

| Item | Verified value |
| --- | --- |
| Source tree | [CachyOS Linux](https://github.com/CachyOS/linux) |
| Source branch | `7.2/cachy` |
| Base commit | [`5c28b6663fbb6d258da53c39d2c8abb30abaafe1`](https://github.com/CachyOS/linux/commit/5c28b6663fbb6d258da53c39d2c8abb30abaafe1) |
| Kernel source version | `7.2.0-rc1` |
| Patch paths | 6 |
| Patch delta | 275 insertions, 2 deletions |

The base commit is the reproducible compatibility boundary. The branch name is
informational and may move after this document is published.

## Changed files

| Path | Change |
| --- | --- |
| `include/linux/buildid.h` | Makes the vmlinux build-ID interface available when the metadata feature is enabled. |
| `init/Kconfig` | Adds the metadata feature, profile, flavor, codename, and revision settings. |
| `kernel/Makefile` | Builds the metadata implementation into the kernel when enabled. |
| `lib/buildid.c` | Provides vmlinux build-ID storage for the metadata feature. |
| `scripts/setlocalversion` | Generates and validates the structured kernel release. |
| `kernel/zeroday0619_kernel_info.c` | Creates the sysfs interface and its read-only attributes. |

## Configuration

| Symbol | Type | Default | Purpose |
| --- | --- | --- | --- |
| `CONFIG_ZERODAY0619_KERNEL_INFO` | Boolean | `y` when `SYSFS` is available | Enables the release format and sysfs interface. |
| `CONFIG_ZERODAY0619_KERNEL_PROFILE` | String | `workstation` | Selects the `workstation` or `server` deployment profile. |
| `CONFIG_ZERODAY0619_KERNEL_FLAVOR` | String | `generic` | Selects the `generic`, `bore`, or `rt` kernel flavor. |
| `CONFIG_ZERODAY0619_KERNEL_CODENAME` | String | `fxsenshi` | Sets the distribution codename. |
| `CONFIG_ZERODAY0619_KERNEL_REVISION` | String | `v1+` | Sets the distribution revision. |

The profile must be exactly `workstation` or `server`. The flavor must be
exactly `generic`, `bore`, or `rt`. The codename and revision must be non-empty
and contain only ASCII letters, digits, periods, underscores, plus signs,
tildes, or hyphens. The compiler family is detected from `CONFIG_CC_IS_GCC` or
`CONFIG_CC_IS_CLANG`.

## Kernel release format

When the feature is enabled, `scripts/setlocalversion` returns:

```text
<kernel-version>-<codename>-<profile>-<flavor>-<compiler>-<revision>
```

For the verified base commit, default metadata, and LLVM, the intended value is:

```text
7.2.0-rc1-fxsenshi-workstation-generic-llvm-v1+
```

With `ibuki`, `workstation`, LLVM, and revision `v1`, the supported flavor
names produce:

```text
7.2.0-ibuki-workstation-generic-llvm-v1
7.2.0-ibuki-workstation-bore-llvm-v1
7.2.0-ibuki-workstation-rt-llvm-v1
```

This path replaces `localversion*`, `CONFIG_LOCALVERSION`, the `LOCALVERSION`
environment variable, and the normal SCM commit or dirty suffix. Builds with
identical metadata can therefore share the same release and module installation
directory even when their source commits differ.

Keep the complete `UTS_RELEASE` within the kernel's 64-character limit.

## Sysfs interface

The patch creates `/sys/kernel/zeroday0619` during late kernel initialization.

| Attribute | Mode | Value |
| --- | ---: | --- |
| `product_name` | `0444` | `zeroday0619 high performance kernel` |
| `kernel_release` | `0444` | Complete `UTS_RELEASE` value |
| `kernel_version` | `0444` | Base major, patchlevel, and sublevel |
| `kernel_codename` | `0444` | Configured codename |
| `kernel_profile` | `0444` | `workstation` or `server` |
| `kernel_flavor` | `0444` | `generic`, `bore`, or `rt` |
| `kernel_buildtype` | `0444` | `gcc` or `llvm` |
| `kernel_revision` | `0444` | Configured revision |
| `system_architecture` | `0444` | `UTS_MACHINE` target architecture |
| `build_uuid` | `0400` | UUID-formatted value derived from the vmlinux build ID |

`build_uuid` copies the first 16 bytes of the vmlinux build ID, then rewrites
the UUID version and variant bits. It is not the original ELF build-ID string.
A non-zero vmlinux build ID is required; otherwise initialization fails and the
sysfs directory is not created.

## Apply

Set `patch_file` to the absolute location of the patch artifact:

```bash
git clone https://github.com/CachyOS/linux.git cachyos-linux
cd cachyos-linux
git switch --detach 5c28b6663fbb6d258da53c39d2c8abb30abaafe1

patch_file=/path/to/kernel-public-patch/patches/0001-kernel-metadata/0001-feat-kernel-expose-zeroday0619-kernel-metadata.patch
git status --short
git apply --check --index --whitespace=error-all "$patch_file"
git switch -c zeroday0619-kernel-info
git apply --index --whitespace=error-all "$patch_file"
git diff --cached --check
git diff --cached --stat
```

`git status --short` must produce no output before the check. Do not use
`--reject`, `--ignore-whitespace`, or `--whitespace=fix` to force application.
The artifact is a unified diff without complete email headers, so use
`git apply`, not `git am`. The [`git apply` documentation](https://git-scm.com/docs/git-apply)
describes the check and index behavior.

## Configure and build

The following x86-64 GCC example uses a separate output directory:

```bash
build_directory="$PWD/build-gcc"

make O="$build_directory" ARCH=x86 CC=gcc x86_64_defconfig
scripts/config --file "$build_directory/.config" \
    --enable SYSFS \
    --enable ZERODAY0619_KERNEL_INFO \
    --set-str ZERODAY0619_KERNEL_PROFILE workstation \
    --set-str ZERODAY0619_KERNEL_FLAVOR generic \
    --set-str ZERODAY0619_KERNEL_CODENAME fxsenshi \
    --set-str ZERODAY0619_KERNEL_REVISION 'v1+'
make O="$build_directory" ARCH=x86 CC=gcc olddefconfig
make -s O="$build_directory" ARCH=x86 CC=gcc kernelrelease
make -j"$(nproc)" O="$build_directory" ARCH=x86 CC=gcc
```

For LLVM, use a different output directory and pass `LLVM=1` to every `make`
invocation, including configuration and `kernelrelease`. Kernel dependencies,
installation, bootloader integration, signing, and initramfs generation are
distribution-specific. Follow the upstream
[kernel build documentation](https://docs.kernel.org/admin-guide/README.html)
and keep a known-good kernel available.

## Runtime verification

After installing and booting the patched kernel:

```bash
sysfs_directory=/sys/kernel/zeroday0619

test -d "$sysfs_directory"
test "$(cat "$sysfs_directory/product_name")" = \
    "zeroday0619 high performance kernel"
test "$(cat "$sysfs_directory/kernel_release")" = "$(uname -r)"
test "$(cat "$sysfs_directory/kernel_profile")" = "workstation"
test "$(cat "$sysfs_directory/kernel_flavor")" = "generic"
test "$(cat "$sysfs_directory/kernel_buildtype")" = "gcc"
test "$(cat "$sysfs_directory/system_architecture")" = "$(uname -m)"
test "$(stat -c '%a' "$sysfs_directory/product_name")" = "444"
test "$(stat -c '%a' "$sysfs_directory/build_uuid")" = "400"
sudo cat "$sysfs_directory/build_uuid"
```

Run this smoke test on a disposable VM or a system with a known-good fallback
kernel. A successful build alone does not verify the late initcall, build-ID
availability, sysfs creation, or file modes.

## Validation status

Validated against base commit
`5c28b6663fbb6d258da53c39d2c8abb30abaafe1`:

- `git apply --numstat` parsed all six paths.
- All five modified-file preimage blob IDs match the base commit.
- `git apply --check --index` passed.
- `git apply --check --index --whitespace=error-all` passed.
- POSIX shell syntax validation passed for `scripts/setlocalversion`.
- All 12 profile, flavor, and compiler combinations produced the expected
  release format in the non-build shell fixture.
- Unsupported flavor validation returned a non-zero status and the expected
  error message.

A full kernel build and runtime boot test have not been performed.
Runtime behavior remains **Verification required**.

## Known limitations

- The sysfs interface has no `Documentation/ABI` entry or dedicated selftest.
- Compiler metadata records the compiler family, not its version or flags.
- Kernel flavor metadata is explicitly configured. It does not verify that a
  matching BORE or PREEMPT_RT implementation is present in the source tree.
- `kernel_version` contains only the base three-component version.
- A failed build-ID or sysfs initialization has no fallback interface.
- No performance benchmark applies because this patch does not change kernel
  performance behavior.

## License

The new kernel source file declares `SPDX-License-Identifier: GPL-2.0-only`.
The patched kernel source remains subject to the applicable upstream Linux
kernel licensing terms. See the upstream
[Linux kernel licensing rules](https://docs.kernel.org/process/license-rules.html).
