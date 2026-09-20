# zeroday0619 high performance kernel public patches

> Public patches maintained for the zeroday0619 high performance kernel distribution. Each patch is stored under [`patches/`](./patches/), with compatibility, behavior, application, and verification details under [`docs/`](./docs/).

## Patch catalog

| Patch | Target | Documentation |
| --- | --- | --- |
| Kernel identity and build metadata | CachyOS Linux `7.2/cachy` at `5c28b66` | [Details](./docs/0001-kernel-metadata.md) |
| Linux 7.2.6 local integration: BORE, acpi_call, DKMS, metadata, Charcoal, video tools | Exact recorded local preimages; not pristine upstream 7.2.6 | [Details](./docs/0002-linux-7.2.6-integration.md) |

## Repository layout

```text
.
├── patches/
│   ├── 0001-kernel-metadata/
│   └── 0002-linux-7.2.6-integration/
│       ├── series          Ordered integration patches
│       └── provenance/     Original inputs, not an application series
├── docs/                   Per-patch documentation
├── LICENSE                 Repository license
└── README.md               Patch catalog and repository overview
```

## Using a patch

1. Open the patch documentation from the catalog.
2. Prepare the documented base commit or exact recorded preimages. A version
   number alone does not identify a compatible base.
3. Run the documented dry-run and static checks.
4. Apply the patch or its ordered `series` on a dedicated branch. Do not apply
   overlapping catalog entries twice; follow the selected guide.
5. Build, boot, and verify the documented behavior before deployment.

Do not assume that a patch applies to a newer revision of the same branch.
Linux kernel patches are context-dependent; failed hunks, offsets, or fuzz
require review and usually a rebase. See the upstream guidance on
[applying kernel patches](https://docs.kernel.org/process/applying-patches.html).

## License

Repository-level material is distributed under the [MIT License](./LICENSE).
Files changed or added within a Linux kernel source tree remain subject to their
declared SPDX identifiers and the applicable upstream Linux kernel licensing
terms.
