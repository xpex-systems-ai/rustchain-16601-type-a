# Claim-by-Claim Sources

Primary upstream repository: https://github.com/Scottcjn/Rustchain

## RIP-PoA identity and purpose
**Claim:** RIP-PoA is the hardware fingerprint attestation layer for RIP-200's 1-CPU-1-Vote consensus.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Abstract and metadata table.

## Hardware checks
**Claim:** The protocol documents clock drift, cache timing, SIMD identity, thermal drift, instruction jitter, device-age evidence, anti-emulation and ROM fingerprint checks.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Sections 2–3 and Reference Implementation.

## Server verification
**Claim:** Raw evidence is validated server-side and the client-reported pass field is not trusted.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Section 3.

## Attestation lifetime
**Claim:** `ATTESTATION_TTL = 86400` seconds (24h).

Source: `specs/RIP_POA_SPEC_v1.0.md`, Sections 2.1 and 12.

## Round-robin producer selection
**Claim:** Valid attested miners are sorted and producer selection uses slot modulo miner count; each miner receives one turn per rotation.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Section 6.1.

## Multiplier examples
**Claim:** PowerPC G4 = 2.5x; Core 2 Duo = 1.3x; Modern ARM = 0.0005 anti-farm penalty.

Sources:
- `specs/RIP_POA_SPEC_v1.0.md`, Section 5.1.
- `docs/vintage-mining/guide.md`, Official Antiquity Multiplier Scale.

## Time-aged decay
**Claim:** Vintage bonuses use a 0.15/year decay factor; base multipliers below 1.0 remain penalties instead of being inflated toward 1.0.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Section 5.2.

## Epoch and reward
**Claim:** 144 × 600-second slots = 24h epoch; documented per-epoch reward is 1.5 RTC.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Sections 6.2 and 12.

## Reward weighting
**Claim:** Failed fingerprint receives weight 0; eligible reward is proportional to time-aged weight.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Section 6.2.

## Threat model
**Claim:** The specification discusses VM farming, emulator spoofing, cloud mining, architecture inflation, timing replay and wallet hopping.

Source: `specs/RIP_POA_SPEC_v1.0.md`, Section 9.

## Active node components
**Claim:** The node README identifies RIP-200 consensus, hardware binding, fingerprint checks, time-aged rewards and Ergo anchoring as key components/features.

Source: `node/README.md`.

## Verification note
This package deliberately avoids unsupported performance benchmarks and speculative earnings claims. Values in the narration are taken from the cited upstream specification/repository.
