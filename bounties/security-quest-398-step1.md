# RustChain Security Quest #398 — Step 1

**Claimant:** @xpex-systems-ai  
**Native RTC wallet:** `RTC82c21b7f32d0e65c4aa9785d6561a55ff6127269`  
**Source reviewed:** `Scottcjn/Rustchain@0906b21bd6d8c7c75c282df86adb2c5bcec88ff6`  
**Scope:** public-source review only. No production exploitation, fund movement, credential access, or destructive testing was performed.

## 1. Attestation and `/attest/submit`

RustChain's attestation path is the boundary between a miner's hardware claim and reward eligibility. The public RIP-PoA specification describes the flow as a client gathering hardware evidence and submitting it to `POST /attest/submit`. The server then validates raw fingerprint evidence through `validate_fingerprint_data()` and derives a canonical verified device through `derive_verified_device()`. That division is security-critical: a client-side claim that a check “passed” is not sufficient evidence by itself, and a self-reported architecture should not be able to select a premium reward tier when measured evidence contradicts it.

The repository also contains attestation clients in the SDK and several miner implementations, but the trust decision belongs on the node. Identity/authenticity and hardware truth are separate questions. A cryptographic signature can bind a request to a key, while fingerprint validation attempts to establish that the request represents the physical device class being claimed. Keeping those controls independent reduces the risk that possession of many software identities automatically becomes many rewarded miners.

Attestation evidence also needs freshness. Hardware identity participates in an epoch-based reward system, so stale evidence should not become a permanent authorization token. The security objective is therefore not simply “accept a valid payload,” but to bind a sufficiently fresh, server-validated hardware observation to the miner identity used for reward eligibility.

## 2. Hardware fingerprinting and VM-farm resistance

The anti-Sybil model is based on physical corroboration rather than trusting a CPU model string. `validate_fingerprint_data()` is documented as server-side validation of raw evidence, and the protocol uses multiple classes of signals rather than relying on one user-controlled field. The RIP-PoA material describes timing/clock behavior, cache characteristics, SIMD identity, thermal behavior, instruction jitter, anti-emulation evidence, and device-age or ROM evidence where applicable.

This matters economically. If a virtual machine could submit a premium vintage architecture string and have that value trusted directly, software instances could manufacture rewarded identities. RustChain instead attempts to make architecture claims agree with measured hardware characteristics and derives the verified device on the server. A failed fingerprint is not intended to receive normal positive reward weight.

The model is strongest when these checks remain independent. Architecture corroboration constrains reward-tier spoofing; timing, cache and thermal signals make emulation more expensive; anti-emulation checks address obvious virtualization; and server-side interpretation prevents a modified client from simply changing a local result from FAIL to PASS. None of those controls alone proves unique physical hardware, but together they raise the cost of a VM-farm strategy and provide multiple points at which contradictory evidence can fail closed.

## 3. Epoch reward calculation and distribution

The current reward implementation is centered on `calculate_epoch_rewards_time_aged()` in `node/rip_200_round_robin_1cpu1vote.py`, with settlement integrated through `node/rewards_implementation_rip200.py`. The public RIP-PoA specification separates block-production turns from reward weighting: eligible attested miners participate in deterministic round-robin selection, while epoch rewards are distributed according to verified/time-aged hardware weight.

The documented model uses a fixed epoch reward pool and computes a weight for each eligible miner. Failed fingerprint evidence produces zero reward weight. For miners that pass, the verified hardware class contributes an antiquity multiplier, and vintage bonuses are time-aged rather than remaining permanently elevated. The reward calculator distributes the epoch pool proportionally to eligible weights.

This means “one CPU, one vote” for producer rotation should not be confused with equal reward amounts when verified hardware antiquity weights differ. The separation is useful defensively because it makes the dependencies explicit: attestation establishes eligibility, verified hardware influences weight, and settlement distributes a bounded pool. A defect in any one of these boundaries can have a different impact and should be tested independently.

## 4. Potential attack vector / hardening target

The attack surface I would prioritize is **sparse-but-valid fingerprint evidence combined with compatibility or fallback behavior**. The repository contains prior security analysis around RIP-201 that illustrates why this class deserves continued attention: validation may be correct for every individual field received while fleet-level or architecture-level classification becomes weaker if too few independent physical signals are comparable.

An adversary would not necessarily submit an obviously invalid fingerprint. A more realistic strategy is to minimize the evidence supplied while remaining inside accepted capability/fallback rules for a claimed device family, then create multiple identities whose remaining fields avoid strong cross-device similarity detection. This is particularly important for genuinely old hardware because the protocol cannot simply require every modern instruction or timer feature without excluding legitimate vintage CPUs.

This assessment does **not** claim that this is a new exploitable vulnerability on current main. It is a threat-model and hardening target. Any compatibility path that relaxes required checks should be bound to a server-derived hardware capability class, require the maximum independent evidence that class can honestly provide, and fail closed when the reason for missing evidence is ambiguous. Fleet/Sybil detection should distinguish “feature genuinely unavailable on this architecture” from “client omitted a feature that would have made instances comparable.”

A useful regression invariant is: **reducing submitted evidence must never increase a miner's reward tier or make a modern/virtualized device easier to classify as premium vintage hardware.** Tests should cover sparse evidence, contradictory architecture/SIMD claims, repeated near-identical fingerprints across identities, and capability-limited genuine vintage profiles.

## Sources reviewed

- `specs/RIP_POA_SPEC_v1.0.md`
- `node/rip_200_round_robin_1cpu1vote.py`
- `node/rewards_implementation_rip200.py`
- `node/rustchain_v2_integrated_v2.2.1_rip200.py`
- `docs/rip201_fleet_detection_bypass.md`
- `docs/rip201_bucket_spoof.md`

**Requested quest credit: Step 1 — 10 RTC, subject to maintainer review.**
