> Historical implementation review for the local 7.2.6 source snapshot. Temporary artifact names below are evidence references, not installation paths. Use the public series and commands in [the integration guide](../../0002-linux-7.2.6-integration.md). Kernel compilation and runtime checks were not performed.

# Charcoal compile and Zen port for Linux 7.2.6

No kernel, module, or userspace program was built. No installation or runtime settings were changed. The root coordinator owns configuration regeneration and complete-series validation.

| Input | Result and source evidence |
|---|---|
| c23.patch (d70f79fef65810faf64dbae1f3a1b5623cdb2345) | Already present in tools/lib/bpf/libbpf.c: const res, sym_sfx, next_path. Full reverse dry-run passed with fuzz zero. Downloaded kernel.org payload SHA256 60824aa965612895dd520c7ae84e75d4c22e11e34c39ab735930a6826601e3bb differs from PKGBUILD. Independently fetched GitHub commit payload SHA256 c1a262ca53fde7d8cd3b7c7479d6c908c73a9996c88e94afed5b71130c9bc788. All three hunks are identical; only Subject decoration, abbreviated index lengths, and cgit trailer differ between those two sources. Expected PKGBUILD bytes were not recovered. |
| 0013-optimize_harder_O3.patch | Already present: Makefile has C -O3, Rust opt-level=3, and cc-option-guarded modulo scheduling; init/Kconfig has the O3 choice. Reverse Kconfig hunk passes; reverse Makefile hunk fails because existing surrounding build code differs. Retained existing implementation. |
| 2990_libbpf-v2-workaround-Wmaybe-uninitialized-false-pos.patch | Applied unchanged at tools/lib/bpf/elf.c; exact forward and reverse dry-run passed. GCC-only diagnostic suppression surrounds elf_find_func_offset_from_file. |
| 5010_enable-cpu-optimizations-universal.patch | Ported 40 missing CPU choices and paired C/Rust compiler flags; preserved existing X86_NATIVE_CPU, MZEN4, GENERIC_CPU, ISA levels 1-4 and native compiler guard. Restricted optimization choice to X86_64, matching the placement of its flags. Preserved modern architecture feature dependencies already implied by X86_64. Added MPSC cache shift. Fixed source patch's invalid Rust -mno-tbm on three AMD choices to -Ctarget-feature=-tbm; corrected Cannon Lake description. |
| dkms-clang.patch | Already present: scripts/Makefile.clang lacks both removed warning errors and the renamed scripts/Makefile.warn lacks strict-prototypes and incompatible-pointer-types errors. No edits needed. |
| 0001-clang-polly.patch | Applied all three sequential component patches with fuzz zero. Strengthened Kconfig cc-option probe to cover every emitted Polly flag, including DCE, so unavailable toolchain options cannot enable the feature. Feature remains optional. |
| cab7ea1a4ef6685a133ae121ca27098b9dd31287.patch | Applied both P-state schedutil select removals. Static search found no schedutil/sugov reference in intel_pstate.c or amd-pstate.c. |
| fb5c79d96cc87e4778ac0f2a53bc7c0c23078c54.patch | Added ZEN_INTERACTIVE, default y, with help describing only the integrated behavior. Existing CACHY and scheduler options preserved. |
| 21dd0495958b7c1bd34f2d83537a4f3af5b804c3.patch | Existing hugepage behavior was gated by CACHY. Extended condition to CACHY OR ZEN_INTERACTIVE, preserving prior behavior. |
| b418708702f7927a7922b90871ab1cdf1df9bb94.patch | COMPACT_UNEVICTABLE_DEFAULT now defaults to zero for PREEMPT_RT OR ZEN_INTERACTIVE. |
| 92850f57d0d3dd0c55a6556f4c4a9afd38da7f8a.patch | Existing zero watermark boost was gated by CACHY. Extended condition to CACHY OR ZEN_INTERACTIVE, preserving prior behavior. |
| e3afdec765f5277bbd3b2196e0facb8b428fb9d2.patch | Already superseded: mm/swap.c unconditionally sets page_cluster=0. Preserved existing behavior rather than reintroducing readahead when ZEN_INTERACTIVE is disabled. |

Static assertions passed for all 40 new CPU-choice/compiler-block mappings, absence of duplicate x86 Kconfig symbols, corrected Rust TBM flags, and byte-identical C23 hunk bodies from two upstream endpoints. The working tree has no .git repository, so a plain git diff --check invocation is unavailable; the coordinator can check the generated consolidated patch instead.
