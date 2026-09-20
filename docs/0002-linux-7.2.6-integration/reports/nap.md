> Historical implementation review for the local 7.2.6 source snapshot. Temporary artifact names below are evidence references, not installation paths. Use the public series and commands in [the integration guide](../../0002-linux-7.2.6-integration.md). Kernel compilation and runtime checks were not performed.

# NAP 0.4.0 port to Linux 7.2.6

Status: Applied `6.16-nap-v0.4.0.patch` with zero fuzz, then adapted the imported source. No NAP implementation existed before application.

Owned paths: drivers/cpuidle/Kconfig, drivers/cpuidle/governors/Makefile, and six files in drivers/cpuidle/governors/nap/.

Changes relative to source patch:
- Removed the backport's duplicate RESIDENCY_THRESHOLD_NS definition and unused gov.h include. Linux 7.2.6 gov.h defines that name as 15 microseconds, whereas the backport locally redefined it as TICK_NSEC. The NAP stop-tick policy now directly uses TICK_NSEC, preserving its intended behavior without a macro redefinition.
- Clamped negative tick_nohz_get_sleep_length() returns to zero at all three NAP call sites. The API documentation in kernel/time/tick-sched.c explicitly permits negative values. Previously the two integer paths converted them to u64, permitting spurious deep-idle selection. The floating-point path now uses the same clamped duration.
- Made CONFIG_CPU_IDLE_GOV_NAP opt-in (no default y) and lowered its rating to 1. Existing governors have ratings at least 9; explicit cpuidle.governor=nap continues to select it. If NAP is the only compiled governor, core selection still selects it normally.
- Corrected Kconfig help to the implemented three 8-to-8-to-1 experts and SSE2/AVX2+FMA, removing inaccurate AVX-512 claims.
- Added an explicit percpu.h include to nap.h. This is header hygiene, not a proven compiler error: cpuidle.h already includes it transitively.
- Preserved existing per-translation-unit LTO removal and FPU flag separation. nap.c remains normal non-FPU kernel code; floating-point entry remains enclosed in kernel_fpu_begin/end and guarded by may_use_simd(). The tree provides CC_FLAGS_FPU and the one-argument timer API required by this source.

Validation:
- Upstream patch dry-run and apply with patch --fuzz=0: passed, all 8 files.
- Generated normalized charcoal-nap-7.2.6.patch against kernel-726-before-charcoal.tar.
- patch --dry-run --reverse --fuzz=0 -p1 < charcoal-nap-7.2.6.patch: passed, all 8 files.
- Whitespace scan of all 8 changed files: passed.
- git diff --check unavailable because the target is not a Git working tree; direct file checks used instead.
- No build, install, governor switch, or runtime setting changes performed.

Verification required: compiler/Kbuild validation, boot, FPU/SIMD behavior, idle residency/latency and performance. This is an experimental imported governor; source integration does not establish runtime safety or performance improvement. Sysfs statistics remain best-effort snapshots, as in the upstream implementation; no full concurrency audit was performed.

Final style pass:
- Fixed all 12 imported checkpatch errors in nap.h (compound-literal spacing and conditional trailing statements).
- Fixed all 22 straightforward comment and declaration-spacing warnings across NAP files.
- scripts/checkpatch.pl --no-tree --terse --file over all five NAP C/header files reports 0 errors and 3 NEW_TYPEDEFS warnings.
- The three vector typedef warnings are retained intentionally: v4sf, v4si, and v8sf name compiler vector types carrying __vector_size__ attributes used throughout the imported SSE2/AVX2 implementation. They are a documented style exception, not a behavioral change.
- nap-checkpatch.log records the result. The aggregate patch must be regenerated after these formatting changes; the earlier standalone normalized patch predates this final style pass.
