> Historical implementation review for the local 7.2.6 source snapshot. Temporary artifact names below are evidence references, not installation paths. Use the public series and commands in [the integration guide](../../0002-linux-7.2.6-integration.md). Kernel compilation and runtime checks were not performed.

# Charcoal network adaptation for Linux 7.2.6

## Per-input disposition

- 2000 Bluetooth: superseded. `net/bluetooth/hci_conn.c:hci_conn_check_link_mode()` no longer enforces minimum key size at all. Existing SSP-only encryption check already permits legacy non-SSP connections. Retained Secure Connections Only and FIPS AES-CCM validation unchanged.
- 302 Minstrel fraction: applied unchanged. Retrieved exact OpenWrt revision 0ff1553bd731c0db28043fc9caab90bdc32587f3 from official GitHub mirror; SHA256 matches.
- 303 Minstrel probability: applied unchanged after 302.
- 304 Minstrel downgrade: adapted CPTCFG_MAC80211_DEBUGFS context to CONFIG_MAC80211_DEBUGFS. Removed max_tp_idx and max_tp_prob declarations/assignments rendered unused by the upstream patch's deleted comparison. Added complete explanatory comment in newly introduced fallback threshold. All functionality retained.
- 910 ath11k CE mapping: applied with zero fuzz and line offsets only. All ATH11K_CE_OFFSET uses removed; IPQ5018 register entries carry CE type bits and AHB accessors route these to mem_ce, masking offset bits.
- 350 ath11k key removal: adapted to set arg.key_len=0 for DISABLE_KEY, retaining explicit WMI_CIPHER_NONE and existing NULL guard around memcpy. WMI_CIPHER_NONE is defined as 0 (clear key) in wmi.h, so explicit assignment equals original patch's zero-initialized struct value. A zero length produces a zero-length WMI array TLV with no key payload. ath11k_wmi_alloc_skb() zeroes the entire allocated body, so no uninitialized tail is transmitted. Retained NULL guard avoids the original patch's memcpy(NULL,0) issue. Firmware behavioral benefit requires hardware validation.
- ath11k-upstream: already integrated, no source change. Raw lore message unavailable (HTTP 403 including escalated request). Official torvalds/linux merged commit e225b36f83d7926c1f2035923bb0359d851fdb73 links the exact requested message ID. All three functional hunks reverse-apply with zero fuzz; copyright-only hunk differs in current source. Current ath11k_dp_rx_ampdu_stop() selects &peer->rx_tid[params->tid], tests rx_tid->active, passes rx_tid to REO update and rx_tid->paddr to queue setup. Original mail SHA256 cannot be verified; merged patch has SHA256 a716d089364a86891403768e2c6970a7d8fa87c11a203ef13f3ae5ddbb362d98.

## Source checksums

| Input | Download SHA256 | PKGBUILD SHA256 |
|---|---|---|
| 2000_BT-Check-key-sizes-only-if-Secure-Simple-Pairing-enabled.patch | 882156f8dfb21b5b1a85e9aaa48280540b4d1348f1bde0c358b47678aea9065a | 882156f8dfb21b5b1a85e9aaa48280540b4d1348f1bde0c358b47678aea9065a |
| 302-mac80211-minstrel_ht-fix-MINSTREL_FRAC-macro.patch | bf2186776d96122136019b7b11aea1f0f46914bf107aa83c949e654290f7eed3 | bf2186776d96122136019b7b11aea1f0f46914bf107aa83c949e654290f7eed3 |
| 303-mac80211-minstrel_ht-reduce-fluctuations-in-rate-pro.patch | 78da5c2c011b2679f1309366c3964a919607db5fa1b76a3e426c5af67eded5a1 | 78da5c2c011b2679f1309366c3964a919607db5fa1b76a3e426c5af67eded5a1 |
| 304-mac80211-minstrel_ht-rework-rate-downgrade-code-and-.patch | 4929f7a8033f34715c2a19b606c45d0d711e7328452ed1b31a5bf52a0c1a7232 | 4929f7a8033f34715c2a19b606c45d0d711e7328452ed1b31a5bf52a0c1a7232 |
| 910-ath11k-fix-remapped-ce-accessing-issue-on-64bit-OS.patch | e261cfdf1d03f741ba111c812f3c1d0be2bf2d58e68efe2477a5bd542cd85f2e | e261cfdf1d03f741ba111c812f3c1d0be2bf2d58e68efe2477a5bd542cd85f2e |
| 350-ath11k-Revert-clear-the-keys-properly-when-DISABLE_K.patch | 49931b2d29f2501bb7d11f0f0cc978d98c90b5556e9ecfe11ca82672445d4cbf | 49931b2d29f2501bb7d11f0f0cc978d98c90b5556e9ecfe11ca82672445d4cbf |
| ath11k-upstream.patch | Unavailable (HTTP 403) | 74db38cd3c353c295d2bd11159ccafc4396b8fb21735a536f5bb5ab71093a90f |

## Validation

- Initial 302/303/910 forward dry runs and application passed with --fuzz=0.
- Adapted 304 forward dry run/application passed with --fuzz=0.
- Aggregate charcoal-network-726.patch created against full pre-Charcoal backup; reverse dry run passed with --fuzz=0 for all seven changed files.
- The workspace has no usable Git history. The coordinator verified the consolidated patch with git apply --check --whitespace=error-all instead of relying on a working-tree diff.
- No build, boot, firmware, throughput or hardware tests performed. Verification required.
- No .config or driver activation changes.

Official merged source: https://github.com/torvalds/linux/commit/e225b36f83d7926c1f2035923bb0359d851fdb73
