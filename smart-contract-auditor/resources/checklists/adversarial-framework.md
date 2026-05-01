# Adversarial Thinking & Attack Framework

Static analysis cannot find logic bugs. This framework guides manual vulnerability discovery by thinking like an attacker. Use Phase 2 documentation — especially Key Invariants and User Flows — as your target list.

---

## Core Questions

For every invariant in the Phase 2 document, ask: **"Can I break this?"**

For every user flow:
- **"How would I extract more value than I put in?"**
- **"What if I call these functions in an unexpected order?"**
- **"What if I call this function with extreme or zero values?"**

For every access control entry:
- **"Can I reach this path without the expected privilege?"**

For every external dependency:
- **"What if the external contract behaves maliciously or returns unexpected values?"**

---

## Logic Bugs Static Analysis Misses

- Business logic errors (wrong fee calculation, wrong rounding direction, off-by-one)
- TOCTOU patterns (check-then-act with state changes possible between check and act)
- Incorrect assumptions about external contracts (fee-on-transfer tokens, rebasing tokens, tokens with callbacks) — see `resources/checklists/non-standard-tokens.md`
- Cross-function invariant violations (consistent within one function, broken across a call sequence)
- Economic exploits (first depositor inflation, sandwich attacks on AMM interactions, flash-loan-amplified manipulation)
- Governance / multi-step attack vectors (voting power manipulation, proposal timing)

---

## Impact Amplification

For every DoS or griefing vector, check whether hard caps or resource limits turn a minor nuisance into a critical block:

- **Does a max-slots / max-count constant exist** (e.g., `MAX_VAULTS = 10`)? If so, permanently blocking one slot may prevent new registrations entirely.
- **Does the protocol lack an admin recovery path** (sweep function, force-unregister, emergency override)? If there is no way to reclaim the blocked resource, the DoS is permanent.
- **Can the grief be executed at negligible cost** (1 WEI, dust amount)? Low attack cost + permanent effect + no recovery = severity escalation.

The combination of all three — hard cap exists, no recovery path, negligible cost to attack — typically escalates a "Low" DoS to a "High" finding.

---

## Modern Attack Patterns

These are commonly missed by both static analysis and surface-level code review:

### Read-Only Reentrancy

Protocol A reads state from Protocol B (e.g., LP token price) while Protocol B is mid-execution. The read returns stale/manipulated values because Protocol B hasn't finalized its state update. Common in lending protocols that price LP tokens by querying pool reserves. Slither reentrancy detectors do NOT catch this.

**What to look for:** Any `view` call to an external contract that returns a value used in pricing, collateral calculation, or access decisions. If the external contract has any function that makes callbacks (token transfers, flash loans), the `view` call can be re-entered.

### Donation Attacks Beyond ERC4626

Any contract using `balanceOf(address(this))` or `address(this).balance` for accounting can be manipulated via direct token transfer or `selfdestruct` (pre-Dencun) ETH injection. This applies to:
- Share pricing (the classic ERC4626 inflation attack)
- Precondition checks ("require balance >= threshold")
- Administrative gates ("only callable if balance is zero")
- Lifecycle functions ("unregister only if no funds remain")

### Permit Front-Running

Attacker front-runs a `permit` + `transferFrom` combo by calling `permit` with the victim's signature first. Victim's tx reverts on the duplicate permit nonce. Safe pattern: wrap `permit` in try/catch.

### Signature Replay Across Chains

EIP-712 signatures missing chain ID in the domain separator are replayable on any chain where the contract is deployed at the same address.

### CREATE2 Address Collision

If a protocol uses `CREATE2` to deploy contracts, an attacker who controls the salt may predict and front-run the address. For `create` + `selfdestruct` + `create2` patterns, an attacker can deploy a different contract at the same address.

### Transient Storage Assumptions

Reentrancy guards using `TSTORE`/`TLOAD` (EIP-1153) clear at end of transaction, but nested calls within the same tx share transient storage. A guard set in one `TSTORE` slot can be bypassed by a callback that uses a different slot or checks in a different order.
