> Historical implementation review for the local 7.2.6 source snapshot. Temporary artifact names below are evidence references, not installation paths. Use the public series and commands in [the integration guide](../../0002-linux-7.2.6-integration.md). Kernel compilation and runtime checks were not performed.

# Charcoal memory patch port for Linux 7.2.6

Source inputs are preserved in `charcoal-inputs`. The source tree is modified; no kernel build, installation, boot, live sysctl change, or performance claim is made.

## ZRAM-IR 1.2

Applied to `drivers/block/zram/zram_drv.c`.

- Preserved Linux 7.2.6 slot APIs and allocation-before-replacement behavior. A failed new allocation does not discard the existing slot.
- Added `vm.zram_recomp_immediate` under `CONFIG_ZRAM_MULTI_COMP`, original default 1 and bounds 0..3.
- Corrected sparse compressor priority handling by using the priority slot bound rather than the count of configured compressors.
- Recorded the actual compressor priority for decompression and subsequent recompression.
- Corrected the original incompressible flag lifetime: publication follows slot cleanup, under its lock; configured but untried compressors remain eligible.
- Added `READ_ONCE` and sysctl registration unwind; preserved `CONFIG_SYSCTL=n` initialization.
- Static priority model passed 512 combinations; source assertions and whitespace checks passed. These are not runtime compression tests.

## Kcompressd unofficial 0.5

Applied to `include/linux/mmzone.h`, `mm/mm_init.c`, `mm/page_io.c`, `mm/swap.h`, and `mm/vmscan.c`. The original change to public `include/linux/swap.h` is replaced by an internal declaration in `mm/swap.h`.

- Adapted to the Linux 7.2.6 `struct swap_iocb **` API and current swap-table accessors; preserved zero-mark updates and architecture swap preparation before queueing.
- Retained `vm.kcompressd=24`, bounded 0..256. Zero disables new offload while already queued work completes.
- Worker infrastructure and sysctl are guarded by `CONFIG_SWAP`.
- Node-local kswapd is the sole producer; queued folios retain their lock and an additional reference.
- Serialized FIFO insertion/removal; a full queue falls back to synchronous processing instead of stealing another queued folio.
- Worker shutdown includes the stop condition in its wait and drains the queue before exit. Producer shutdown precedes consumer shutdown and queue free.
- Published only successful, started workers using release/acquire ordering and a completion handshake. Allocation/thread failures leave synchronous reclaim available.
- Rechecked memcg writeback permission at deferred execution time, preserving the 7.2.6 RCU protection. Denied writeback restores dirty state, clears reclaim state, unlocks, and releases the reference.
- Separate read-only lifecycle/API review found no concrete remaining defects. The handshake is defensive hardening, not a claim of a reproduced hot-remove bug.
- Generated `charcoal-kcompressd-7.2.6.patch`; zero-fuzz reverse dry run passed on all five modified paths.

## Re-swappiness 1.2

Applied all four files with `patch --fuzz=0`: `include/linux/mm_inline.h`, `include/linux/mmzone.h`, `mm/vmscan.c`, `mm/workingset.c`. Independent review found no further concrete defect. See detailed appendix below.

## Verification boundary

Build, CONFIG_SWAP/CONFIG_LRU_GEN/CONFIG_ZRAM_MULTI_COMP configuration matrix, boot, memory pressure, swapoff/hot-remove, compression data integrity, and performance remain untested. Human review and environment-specific runtime verification are required before deployment.

## Final MGLRU integration appendix

# Re-swappiness port to Linux 7.2.6

Source: `charcoal-inputs/re-swappiness-v1.2_backported.patch`.
Artifact: `charcoal-mglru-7.2.6.patch`.
Baseline: `charcoal-memory-before`.
All four files were applied by the integrating agent after independent source review.

## Compatibility changes

- Preserve const lruvec signatures, folio flags.f layout, and current lruvec locking helpers.
- Reuse the existing workingset shadow file flag and retain EVICTION_MASK_ANON and mem_cgroup_from_private_id.
- Convert 7.2.6 memcg nowalk generation advancement and reparenting to independent anonymous/file sequences.
- Preserve the refactored 7.2.6 reclaim/isolation implementation and apply type-specific aging at its current call site.
- Omit unused local variables and obsolete scan-control hunks from the input patch.

## Correctness repairs to the input patch

- Separate Bloom filters and MM history counters by type, including filter teardown.
- Permit private file mappings containing anonymous COW folios during anonymous aging. Filter leaf PTE and huge PMD folios by their actual LRU type before clearing young bits or promoting them.
- Restrict rmap neighborhood promotion to the triggering folio type, so adjacent COW folios cannot be promoted using the wrong type's generation.
- Stop clearing shared mm activity bitmap bits. Both independent walkers need the same activity hint.
- Disable the shared nonleaf PMD young-bit optimization. Leaf PTE and huge PMD young-bit checks remain active. The enabled sysfs mask no longer advertises the NONLEAF_YOUNG bit.
- Display only the bounded union of retained type-specific sequence intervals in debugfs. Skip potentially arbitrarily large sequence gaps and suppress stale ring-slot aliases from the other type.
- Keep run_aging's existing behavior of advancing the current generation when supplied an older acceptable sequence.

The shared-hint changes trade potentially higher page-table scanning cost for avoiding lost activity observations. Performance improvement is not established.

## Validation

- `patch -p1 --dry-run --fuzz=0 < charcoal-mglru-7.2.6.patch`: passed against the live tree; mmzone hunks used a two-line offset from the integrating agent's includes.
- Static regular-expression audit: no remaining scalar field access to lrugen.max_seq, mm_state head/tail/seq, or walk seq; all Bloom filters and timestamps use both indexes.
- Repository-wide reference search found affected max_seq and lru_gen_is_active code references confined to the four owned files.
- Exhaustive small-state model: 1,458 combinations of divergent anonymous/file sequences and debug modes produced the exact union of retained sequence intervals, with at most eight output generation rows.
- `git diff --no-index --check` for vmscan: no whitespace diagnostics (exit 1 reflects the presence of differences).
- No build, boot, or runtime reclaim/performance test was performed. Compilation and runtime behavior require verification.

## Combined artifact

`charcoal-memory-7.2.6.patch` contains all eight changed files relative to the exact pre-change baseline. Original input patches remain intact. No Signed-off-by was added. Checkpatch on the intermediate kcompressd artifact reported missing commit metadata/signoff and barrier-comment warnings; source barrier comments were clarified. This port artifact is not an upstream submission.

Combined reverse zero-fuzz dry run passed for all eight files. A fresh baseline scratch application also passed with zero fuzz and matched the live source byte-for-byte in all eight files.

## Follow-up: sysctl bound qualifier fix

Clang 22 rejected the kcompressd upper-bound initializer because
`struct ctl_table.extra2` is `void *` while the bound has type `const int`.
The initializer now uses `(void *)&kcompress_fifo_limit`, matching existing
sysctl constants in this kernel. The constant and its 256 limit remain
unchanged. `proc_dointvec_minmax` reads these bounds and writes only table data.
No warning suppression was added.

The affected translation unit passed:

```sh
make LLVM=-22 LLVM_IAS=1 CC="ccache clang-22" -j2 mm/vmscan.o
```

The resulting object is LLVM IR bitcode under Full LTO. This is a targeted
compile check, not a complete kernel image/module build or runtime test.
The other 46-path port's sysctl bound initializers were checked for the same
uncast-constant pattern; no additional instance was found.
