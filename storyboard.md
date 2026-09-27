# Storyboard — Proof of Antiquity

Target runtime: ~5:15. This document is a capture plan, not a record of terminal output. Any terminal/code shown in the final video must be captured verbatim from the cited public RustChain repository or a real execution.

| Time | Shot | Exact capture / production instruction | Source anchor |
|---|---|---|---|
| 00:00–00:15 | Physical CPU vs VM identities | Original diagram: one physical CPU on left; multiple VM silhouettes on right. Label as conceptual illustration. | `specs/RIP_POA_SPEC_v1.0.md` §1.1 |
| 00:15–00:38 | 1 CPU / 1 Vote problem | Show the RIP-PoA spec Abstract in the public RustChain repo; highlight “distinct physical CPU” and “1-CPU-1-Vote”. | RIP-PoA Abstract |
| 00:38–01:05 | Attestation flow | Original flow diagram: Miner Client → POST /attest/submit → validate_fingerprint_data() → derive_verified_device() → epoch enrollment. | RIP-PoA §2.1 |
| 01:05–01:28 | Fingerprint signals | Eight original cards: clock drift, cache timing, SIMD, thermal drift, instruction jitter, device age, anti-emulation, ROM fingerprint. Do not show invented measurements. | RIP-PoA §3 |
| 01:28–02:05 | Deterministic rotation | Original ring diagram of attested miners sorted by miner ID; animate slot pointer moving one position. Caption: conceptual visualization of documented selection rule. | RIP-PoA §6.1 |
| 02:05–02:38 | Antiquity multiplier examples | Capture the actual multiplier table/code lines from the public repository. On-screen callouts only for values verified in source: PowerPC G4 2.5x, Core 2 Duo 1.3x, modern ARM 0.0005. | RIP-PoA §5.1 |
| 02:38–03:03 | Time-aged bonus | Original chart illustrating bonus-above-1.0 decay. Clearly label “illustration of documented formula”, not measured performance. | RIP-PoA §5.2 |
| 03:03–03:28 | Epoch structure | Original timeline: 144 slots × 600 seconds = 24 hours; show documented 1.5 RTC epoch reward. | RIP-PoA §6.2 |
| 03:28–03:50 | Reward settlement | Original formula card: miner eligible weight / total eligible weight × epoch reward. Note failed fingerprint → zero reward weight according to spec. | RIP-PoA §6.2 |
| 03:50–04:20 | Cross-validation | Split-screen conceptual graphic: claimed architecture vs SIMD/cache/timing/thermal evidence. No claim that any single signal is impossible to spoof. | RIP-PoA §§3–4 |
| 04:20–04:35 | Threat model | Original icons for VM farming, emulator spoofing, cloud mining and wallet hopping. | RIP-PoA §9 |
| 04:35–05:15 | Source-first conclusion | Screen capture of public RustChain repository and this package's SOURCES.md. End card: “Verify the protocol in the source.” | public RustChain repo + SOURCES.md |

## Capture integrity rules
- Never label reconstructed, generated, or reformatted output as a real terminal capture.
- If terminal output is used, run the public command on real hardware and capture its output verbatim; trimming must be disclosed.
- Every numeric protocol claim must match the cited public source.
- Do not present antiquity multipliers as benchmark speed, fiat value, guaranteed profit, or guaranteed earnings.
- Generated diagrams are illustrative and must be labeled as such where a viewer could mistake them for measured output.
