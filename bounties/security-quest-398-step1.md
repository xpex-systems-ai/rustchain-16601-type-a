# RustChain Security Quest #398 — Step 1 (revised after maintainer review)

**Claimant:** @xpex-systems-ai  
**Native RTC wallet:** `RTC82c21b7f32d0e65c4aa9785d6561a55ff6127269`  
**Pinned source reviewed:** `Scottcjn/Rustchain@a4d39e9604897caf30693b05a79181018521811f`  
**Scope:** public-source code review only. No production exploitation, credential access, fund movement, or destructive testing.

This revision responds directly to Elya's review on #398: it follows the real node code path with pinned line references and replaces the prior docs-derived hardening idea with a code-derived attack surface.

## 1. Challenge issue and miner binding

The live flow starts at `POST /attest/challenge` in `node/rustchain_v2_integrated_v2.2.1_rip200.py:6022-6100`. The node rate-limits by the trusted client IP at 6029-6038, generates a 32-byte random challenge at 6040, and gives it a five-minute lifetime at 6041. If the caller supplies `miner` or `miner_id`, the node resolves that identity in the same order later used by submit (6081-6085), then stores `nonce, expires_at, bound_miner` in the `nonces` table at 6087-6093.

The binding is enforced by `attest_validate_challenge()` at 814-848. It selects the live nonce and its `bound_miner` (826-834). If a bound challenge is presented by a different or missing miner, it returns `nonce_identity_mismatch` without deleting the challenge (835-839). A valid challenge is deleted atomically by nonce and expiry at 841-847. That design both prevents cross-identity reuse and avoids letting an attacker burn another miner's bound nonce merely by submitting it under the wrong identity.

## 2. /attest/submit: signature rules, nonce consumption, and replay rejection

`POST /attest/submit` is at 6372-6388 and delegates to `_submit_attestation_impl()` beginning at 6391. The request must be a JSON object (6393-6400), then the node normalizes miner, report, nonce and device at 6408-6412.

The signature logic begins at 6414. The code supports the current canonical-JSON scheme and a legacy four-field scheme (6414-6423). Signature and public-key types are checked before normalization (6425-6446). The enforcement phase is obtained through `_attest_enforce_mode()` (408-416): `log_only`, `enforce_new`, or `enforce_all`. Critically, a partial signature pair cannot silently fall through to the unsigned path: if exactly one of signature/public_key is present, the request is rejected as `INCOMPLETE_SIGNATURE` at 6458-6467. In enforcing phases, identity-state read failure is fail-closed (6469-6485), and unsigned miners are evaluated against allowlist, stored-key, grandfathering and enforcement-mode state (6487 onward).

Nonce replay is separately enforced. `attest_validate_and_store_nonce()` at 851-884 first checks `used_nonces` and returns `nonce_replay` if already seen (864-870). It then calls `attest_validate_challenge(... required_miner=miner)` (872-876), and on success records the nonce with miner and expiry in `used_nonces` (878-883). The submit path invokes this at 6700-6706 and maps identity mismatch to 403 and stale/replayed challenges to 409 at 6707-6737. Thus possession of a previously accepted nonce is not enough to replay an attestation.

## 3. Hardware binding and fingerprint decision

After nonce acceptance, the node runs the wallet-review gate and hardware binding. At 6746-6768, serial-bearing clients can use binding v2; otherwise the legacy path calls `_check_hardware_binding()` at 6770-6787. The legacy binding function is defined at 6187 onward and derives a stable hardware key from server-observed client IP plus device model/arch/family/cores and MAC evidence; the construction is shown at 6104-6135. A hardware key already bound to another wallet is rejected.

Fingerprint validation is explicitly fail-closed for reward purposes. `fingerprint_passed` starts false at 6802-6804. `validate_fingerprint_data(fingerprint, claimed_device=device)` is called at 6806-6812. The replay-defense layer then checks fingerprint replay, entropy collisions, and a hardware-ID rate limit at 6814-6868. A detected replay returns 409 at 6895-6903. Even after the fingerprint validator passes, the server-side VM signature check can force `fingerprint_passed=False` at 6922-6926. The final state is stored by `record_attestation_success(... fingerprint_passed ...)` at 6948-6959.

## 4. How attestation state fixes epoch-enrollment weight

The attestation path auto-enrolls at 7004 onward. It derives a server-verified device at 7015, derives the reward device using both `fingerprint_passed` and canonical measurement verification at 7016-7021, then selects hardware weight at 7022-7029. If the fingerprint failed, the enrollment receives the failed-fingerprint weight; otherwise the validated weight is converted to units (7063-7067). The epoch row is written with `INSERT OR IGNORE` at 7072-7078, making the first enrollment for that miner/epoch win rather than allowing a later low-weight write to overwrite it.

The explicit `POST /epoch/enroll` path at 7237 onward also ties enrollment identity to the most recent attestation signing key: it loads `miner_attest_recent.signing_pubkey` (7282-7299), checks key equality and verifies the signed `miner_pubkey|miner_id|epoch` message (7300-7318), and normally rejects missing ownership proof (7319-7378). It then calls `check_enrollment_requirements()` (7387-7397). Most importantly, reward device data comes from stored attestation state via `resolve_enroll_weight_device()`, not the caller's request body (7400-7409). Fingerprint failure forces failed-fingerprint weight (7454-7460), and the row again uses first-write-wins `INSERT OR IGNORE` (7468-7479).

So the security boundary is concrete: a request-body claim alone does not set premium epoch weight. The weight is downstream of stored attestation/fingerprint state, temporal checks and rotating checks.

## 5. Code-derived attack surface: shared-IP coupling in the legacy no-serial path

A hardening target visible directly in current code is the **legacy hardware-binding key's dependence on source IP**.

`_compute_hardware_id()` states that the key includes source IP and notes that miners behind the same NAT share an IP binding pool (6104-6135). Its key material is `[ip_component, model, arch, family, cores, mac_str]` (6121-6131). When a miner does not enter the serial-bearing binding-v2 branch, `/attest/submit` falls back to this legacy binding at 6770-6772.

The node is careful about where that IP comes from: `client_ip_from_request()` only honors `X-Real-IP` when the direct peer is allowlisted as a trusted reverse proxy, otherwise it uses `remote_addr` (582-600). But the code itself documents an operational boundary: if a reverse proxy runs on another host and `RC_TRUSTED_PROXY_IPS` is not configured, every client collapses onto the proxy IP (591-593). Challenge issuance is also rate-limited by that same trusted client IP: `check_challenge_rate_limit()` keys its counter on `client_ip` at 5287-5335.

**Security/availability consequence:** this is not a claim of a current remote bypass. It is a deployment-sensitive coupling. On a misconfigured proxy deployment, unrelated miners can share the same challenge-rate-limit identity and the same IP component in legacy hardware binding. That can create false-positive throttling and, when otherwise similar legacy/no-MAC device tuples collide, false hardware-binding conflicts. Behind ordinary NAT, the comment at 6121-6123 explicitly accepts some shared-IP coupling as a tradeoff.

**Suggested hardening:** expose a startup/readiness check that warns or fails deployment when requests arrive through an untrusted proxy peer while a proxy topology is expected; add regression tests for multi-client NAT/proxy cases; and progressively retire the legacy no-serial binding path in favor of server-verifiable stable hardware evidence. Rate limiting can remain IP-aware for abuse control, but identity/hardware uniqueness should avoid depending on network topology where stronger verified device evidence is available.

**Severity:** Low / hardening & availability risk. I am not asserting a new exploitable vulnerability or requesting Step 3 credit for this observation.

## Sources

Pinned code: `Scottcjn/Rustchain@a4d39e9604897caf30693b05a79181018521811f`

- `node/rustchain_v2_integrated_v2.2.1_rip200.py:408-416` — attestation enforcement modes
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:582-600` — trusted client-IP derivation
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:814-884` — challenge binding, consume, used-nonce replay rejection
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:5287-5335` — challenge IP rate limit
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:6022-6100` — challenge issuance
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:6104-6135, 6187+` — legacy hardware identity/binding
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:6372-6959` — submit, signature, nonce, hardware and fingerprint gates
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:7004-7078` — attestation auto-enrollment
- `node/rustchain_v2_integrated_v2.2.1_rip200.py:7237-7479` — explicit epoch enrollment and stored-state weighting
- `node/hardware_fingerprint_replay.py:384-459` — hardware-ID fingerprint submission rate limit

**Requested quest credit:** Step 1 — 10 RTC, subject to maintainer re-review.