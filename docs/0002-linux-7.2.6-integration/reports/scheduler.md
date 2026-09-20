> Historical implementation review for the local 7.2.6 source snapshot. Temporary artifact names below are evidence references, not installation paths. Use the public series and commands in [the integration guide](../../0002-linux-7.2.6-integration.md). Kernel compilation and runtime checks were not performed.

# Charcoal scheduling and block patch review for Linux 7.2.6

| Input patch | Disposition | Evidence |
| --- | --- | --- |
| 0001-linux6.16.12-ADIOS-3.1.8.patch | Retain existing newer ADIOS 3.3.0; do not downgrade. | block/adios.c:29; block/Makefile:28; block/elevator.c:743. |
| 0002-Make-ADIOS-the-Default-I-O-scheduler.patch | Applied unchanged with fuzz zero. | block/Kconfig.iosched:21 and :29 now default y. |
| 0001-linux6.16.0-bore-6.5.2.patch | Retain existing BORE 6.8.0 integration. | include/linux/sched/bore.h:14; kernel/sched/bore.c; kernel/fork.c:2456; kernel/futex/waitwake.c:392-396. |
| 0002-sched-ext-coexistence-fix.patch | Superseded by current BORE reweight implementation; obsolete global helper has no callers. | kernel/sched/bore.c:68-81 directly calls reweight_entity and sets inverse weight. kernel/sched/fair.c:4759 exports reweight_entity. |
| 6.16-poc-selector-v2.6.1.patch | Retain existing POC 2.6.3 integration. | kernel/sched/poc_selector.c:53; kernel/sched/fair.c:8987; kernel/sched/ext/ext.c:6367 and :7584; kernel/sched/idle.c:310 and :368; kernel/sched/topology.c:3174. |

## Functional comparison

Extracted complete old added files and compared against current implementations, beyond version labels. All 19 matched single-line BORE function definitions and all 17 matched POC function definitions still have definitions or usages in their respective current implementations. This is a supporting inventory, not proof of runtime equivalence.

ADIOS request insertion, dispatch, completion, adaptive latency models, sysfs attributes and elevator registration remain present. The old adios_init_hctx helper is intentionally absent because the current API uses request_queue-based depth_updated (block/adios.c:834), called during initialization at :1532. The older release_barrier_requests helper and queue-wide barrier state are absent; current implementation documents why generic block flush sequencing owns these guarantees (block/adios.c:62-80). Reintroducing the old hook or barrier code would reverse existing API adaptations and design changes.

BORE retains burst update/restart, inheritance cache, effective priority and sysctl support. Current implementation adds static keys and updated inheritance handling. Futex waiting markers and fork initialization remain integrated. The sched-ext coexistence patch only adds a reweight_task helper; the current BORE implementation directly reweights CFS entities and updates inverse weight, while kernel scheduler classes have separate reweight_task callbacks. Adding the unused historical wrapper offers no missing behavior.

POC retains shared LLC state, idle-state notification, CPU selection and sched_ext activation/deactivation hooks. Its topology initialization is now in poc_sd_shared_init, invoked from topology.c. Both sched_ext notifications are preserved in the relocated ext/ext.c. The four-argument select_idle_sibling interface and POC fallback remain in fair.c.

## Applied change and verification

Only block/Kconfig.iosched was modified by this review. Both default values changed to y, matching the exact Charcoal patch. No .config or runtime scheduler was changed.

- `patch -p1 --fuzz=0 --batch < charcoal-inputs/0002-Make-ADIOS-the-Default-I-O-scheduler.patch`: passed without offsets or fuzz.
- Reverse dry run with `--fuzz=0 --dry-run --reverse --batch`: passed.
- No kernel build, installation, boot or runtime performance validation performed.
