# Narration Script — Proof of Antiquity

Estimated runtime: ~5 minutes.

## 00:00 — The problem
Most blockchain consensus systems tie influence to something abstract: computing work or capital. RustChain takes a different path. Its RIP-200 design starts from a simple rule: one physical CPU, one turn in a deterministic round-robin schedule. But that creates an immediate problem. If the network cannot distinguish a real processor from a virtual machine or emulator, one operator could manufacture many fake identities.

That is where Proof of Antiquity, or RIP-PoA, enters.

## 00:38 — What Proof of Antiquity does
RIP-PoA is RustChain's hardware fingerprint attestation layer for RIP-200. A miner submits evidence derived from physical hardware behavior. The server validates the raw measurements instead of trusting a client-side “passed” flag.

The public specification describes checks covering clock drift, cache timing, SIMD identity, thermal behavior, instruction jitter, device-age evidence, anti-emulation signals and, for applicable retro systems, ROM fingerprints.

A successful attestation is valid for twenty-four hours. The miner can then participate in the current epoch.

## 01:28 — One CPU, one vote
RIP-200 uses deterministic round-robin block production. Attested miners are sorted by miner ID, and the producer for a slot is selected from that list using the slot number modulo the number of attested miners.

That means producer selection is not described as a hash-power lottery or a stake-weighted lottery. Each eligible miner gets one turn in the rotation.

Hardware attestation matters because without it, virtual machines could multiply apparent participants.

## 02:05 — Why old hardware matters
RustChain also assigns reward weights using architecture-specific antiquity multipliers.

The current public RIP-PoA specification lists examples including a PowerPC G4 at a 2.5-times base multiplier and a Core 2 Duo at 1.3 times. Modern ARM is assigned a 0.0005 anti-farm penalty. The specification makes an important distinction: multipliers below one are penalties and do not decay upward toward one.

Vintage bonuses do decay over the lifetime of the chain. The published formula uses a fifteen-percent-per-year decay factor on the bonus above one, eventually converging vintage bonus multipliers toward one.

So the protocol is not simply saying “old computer equals more money forever.” It combines verified hardware identity with a time-aged reward policy.

## 03:03 — How rewards are settled
The specification defines an epoch as 144 slots, with each slot lasting 600 seconds. That makes an epoch twenty-four hours.

The documented per-epoch reward is 1.5 RTC. During settlement, miners with failed fingerprints receive zero reward weight. For miners that pass, the system calculates a time-aged multiplier, sums all eligible weights, and distributes the epoch reward proportionally.

This creates two different ideas that should not be confused: block-production turns are round-robin, while epoch reward distribution is weighted by verified hardware antiquity.

## 03:50 — Fighting spoofing
Proof of Antiquity does not rely on a CPU model string alone. The specification explicitly says self-reported architecture is not trusted by itself.

For example, SIMD capabilities can be cross-checked against the architecture a miner claims. Timing behavior, cache hierarchy, thermal drift and instruction jitter provide additional physical signals. The threat model also covers VM farming, emulator spoofing, cloud mining and wallet hopping.

The design is fail-closed in several important places: missing fingerprint evidence can fail validation, unknown architecture falls back to a non-premium default multiplier, and server-side validation rechecks raw evidence.

## 04:35 — The bigger idea
The interesting idea behind RustChain is not that obsolete hardware magically becomes powerful. It is that physical diversity itself can become part of a consensus and reward system.

RIP-200 defines the rotation. RIP-PoA tries to establish that each participant corresponds to real hardware. Antiquity multipliers then influence reward allocation according to verified architecture and chain age.

Whether this approach succeeds at scale is something deployment and adversarial testing have to demonstrate. But the public implementation gives us something concrete to inspect: the protocol rules, validation checks, multiplier table and settlement algorithm are all documented in the RustChain repository.

If you want to verify every technical statement in this video, the accompanying production package includes a claim-by-claim source map pointing directly to the public RustChain files.
