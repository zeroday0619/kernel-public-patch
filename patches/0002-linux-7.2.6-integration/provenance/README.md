# Source provenance

These inputs are archival data, not the application series. Apply only the six
files listed in the parent directory's `series` file. The original PKGBUILD,
README, signatures, and patch messages retain upstream wording and attribution.
Their SteamOS installation commands, config choices, and performance claims do
not describe this port and were not executed or adopted wholesale.

`local-inputs/` contains the original BORE, acpi_call, DKMS, and metadata inputs.
`charcoal-inputs/` contains the retrieved Charcoal kernel inputs, its original
recipe and README, and two external-module diffs excluded from application.
`charcoal-manifest.json` records all 35 Charcoal input dispositions and checksums.
33 payloads match the recipe exactly. The C23 payload has different formatting,
and the requested ath11k email could not be recovered; both fixes are already
present and were verified against official upstream equivalents without adding
them again. See the integration guide for details.

The normalized patches preserve the resulting source, including existing
whitespace in the BORE help text. No new human sign-off was fabricated.

## BORE upstream verification

[bore-upstream.json](bore-upstream.json) records the branch URL, immutable source
revision, checksum, comparison time, and live source check. The requested
CachyOS 7.2 patch is byte-identical to the archived BORE 6.8.0 input. The
published patch retains the existing Linux 7.2.6 context adaptation. Consult
the recorded check time before treating a moving branch as unchanged.
