# Per-input porting status

Source recipe: `dfe38f01a107e8f15725fb053754f612acb72ed9`, in PKGBUILD application order.

24 inputs contribute to the consolidated source diff. The other 11 are already present or superseded. Source-file checksum matching does not imply build or runtime validation.

| Order | Kernel patch input | Disposition | Original checksum |
| ---: | --- | --- | --- |
| 1 | `vangogh_allow_higher_cpu_freq.patch` | Applied | Matches |
| 2 | `vangogh_higher_max_power_limit.patch` | Applied | Matches |
| 3 | `drm_sched_rr_default.patch` | Ported and applied | Matches |
| 4 | `re-swappiness-v1.2_backported.patch` | Ported and applied | Matches |
| 5 | `0001-linux6.16.12-zram-ir-1.2.patch` | Ported and applied | Matches |
| 6 | `c23.patch` | Already present; retained | Not reproduced; upstream equivalent reviewed |
| 7 | `0002-clear-patches.patch` | Ported and applied | Matches |
| 8 | `0007-v6.11-fsync1_via_futex_waitv.patch` | Ported and applied | Matches |
| 9 | `0013-optimize_harder_O3.patch` | Already present; retained | Matches |
| 10 | `2000_BT-Check-key-sizes-only-if-Secure-Simple-Pairing-enabled.patch` | Superseded; retained newer code | Matches |
| 11 | `2990_libbpf-v2-workaround-Wmaybe-uninitialized-false-pos.patch` | Applied | Matches |
| 12 | `5010_enable-cpu-optimizations-universal.patch` | Ported and applied | Matches |
| 13 | `dkms-clang.patch` | Already present; retained | Matches |
| 14 | `0001-clang-polly.patch` | Ported and applied | Matches |
| 15 | `0001-always-print-firmware-file-name.patch` | Ported and applied | Matches |
| 16 | `302-mac80211-minstrel_ht-fix-MINSTREL_FRAC-macro.patch` | Applied | Matches |
| 17 | `303-mac80211-minstrel_ht-reduce-fluctuations-in-rate-pro.patch` | Applied | Matches |
| 18 | `304-mac80211-minstrel_ht-rework-rate-downgrade-code-and-.patch` | Ported and applied | Matches |
| 19 | `910-ath11k-fix-remapped-ce-accessing-issue-on-64bit-OS.patch` | Applied | Matches |
| 20 | `350-ath11k-Revert-clear-the-keys-properly-when-DISABLE_K.patch` | Ported and applied | Matches |
| 21 | `ath11k-upstream.patch` | Superseded; retained newer code | Not reproduced; upstream equivalent reviewed |
| 22 | `0001-linux6.16.12-ADIOS-3.1.8.patch` | Newer implementation retained | Matches |
| 23 | `0002-Make-ADIOS-the-Default-I-O-scheduler.patch` | Applied | Matches |
| 24 | `0001-linux6.16.0-bore-6.5.2.patch` | Newer implementation retained | Matches |
| 25 | `0002-sched-ext-coexistence-fix.patch` | Superseded; retained newer code | Matches |
| 26 | `0001-linux6.16-kcompressd-unofficial-0.5.patch` | Ported and applied | Matches |
| 27 | `f6ed65cd7bda9cb6009c6a12efd7c4311df31936.patch` | Already present; retained | Matches |
| 28 | `cab7ea1a4ef6685a133ae121ca27098b9dd31287.patch` | Applied | Matches |
| 29 | `fb5c79d96cc87e4778ac0f2a53bc7c0c23078c54.patch` | Applied | Matches |
| 30 | `21dd0495958b7c1bd34f2d83537a4f3af5b804c3.patch` | Ported and applied | Matches |
| 31 | `b418708702f7927a7922b90871ab1cdf1df9bb94.patch` | Applied | Matches |
| 32 | `92850f57d0d3dd0c55a6556f4c4a9afd38da7f8a.patch` | Ported and applied | Matches |
| 33 | `e3afdec765f5277bbd3b2196e0facb8b428fb9d2.patch` | Already present; retained | Matches |
| 34 | `6.16-poc-selector-v2.6.1.patch` | Newer implementation retained | Matches |
| 35 | `6.16-nap-v0.4.0.patch` | Ported and applied | Matches |

## External module inputs

| Input | Disposition |
| --- | --- |
| `ryzen_smu.diff` | Targets external `ryzen_smu` sources; not applied to the kernel tree. |
| `xpad-noone.diff` | Targets external `xpad-noone` sources; not applied to the kernel tree. |

## Detailed adaptations

- [Memory](reports/memory.md): MGLRU, immediate recompression, and deferred compression.
- [Idle](reports/nap.md): NAP and FPU/timer integration.
- [Scheduler and block](reports/scheduler.md): newer existing implementations and ADIOS defaults.
- [Compiler and Zen](reports/build-zen.md): configuration/compiler choices and existing fixes.
- [Network](reports/network.md): Wi-Fi changes and superseded upstream fixes.
- [Clear Linux and futex](reports/clear-futex.md): tuning tradeoffs and compatibility ABI.

Additional root-reviewed changes: DRM policy default and parameter help now agree on round-robin; firmware filename logging follows current filename validation; the two Van Gogh patches preserve their supplied limit values. Existing evdev deferred RCU destruction passed a complete zero-fuzz reverse check, so no duplicate edit was made.
