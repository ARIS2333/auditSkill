# Security Code Reading Checklist

Apply every item to each function flagged by the Phase 2 hypothesis list and Phase 3 detectors. Work through systematically — don't skip items. Record observations in `audit-output/phase-4-analysis/code-reading-notes.md`.

---

## Access Control & Authorization

1. **Inline access control** — `require(msg.sender == owner)` won't show in the `modifiers` printer. Check every function's body for auth patterns not captured by structural analysis.

2. **`tx.origin` vs `msg.sender`** — `tx.origin` used for authorization allows phishing-style relay attacks through intermediate contracts.

3. **Initializer safety** — `initialize()` callable multiple times? Missing `initializer` modifier? Missing `_disableInitializers()` in constructor of implementation contract?

## Reentrancy

4. **Classic reentrancy** — external call before state update? Check the ordering of every `.call`, `.delegatecall`, `.transfer`, `.send`, and interface call relative to state writes.

5. **Cross-function reentrancy** — shared state modified by one function, read by another, with an external call in between. Trace the call graph for shared state variables.

6. **Read-only reentrancy** — Protocol A reads state from Protocol B while Protocol B is mid-execution. The read returns stale/manipulated values. Slither does NOT catch this because the reentrant call is a `view` function.

## Arithmetic & Precision

7. **Integer overflow/underflow** — `unchecked` blocks, type casting (`uint256` → `uint128`), arithmetic near type boundaries.

8. **Precision loss** — division before multiplication, rounding direction (deposits should round DOWN for shares, withdrawals should round DOWN for assets — always favor the vault/protocol), dust amounts, rounding accumulation over many operations.

## Value Extraction & MEV

9. **Front-running / MEV** — sandwich potential on swaps or large state changes? Missing commit-reveal? Slippage parameter user-controllable or hardcoded?

10. **Oracle manipulation** — can price feeds be manipulated within a single transaction via flash loans? Check what oracle is used, what the staleness window is, and whether spot prices are used instead of TWAPs.

11. **Flash loan vectors** — can borrowed capital exploit state in one transaction? Any balance-weighted decisions that are manipulable?

## Denial of Service

12. **DoS vectors** — unbounded loops over user-controlled arrays, grief via revert callbacks, gas griefing via return data bombs, external calls to untrusted addresses in loops.

13. **Donation / direct transfer attacks** — search for EVERY use of `balanceOf(address(this))`, `address(this).balance`, or any balance/amount-based precondition check across ALL functions (not just accounting/pricing). Donation griefing applies wherever a balance check gates a state transition — including administrative and lifecycle functions (registration, unregistration, migration, shutdown, parameter updates). Ask: "Can anyone send 1 WEI to permanently block this code path?"

## External Interactions

14. **External calls** — enumerate all `.call`, `.delegatecall`, `.transfer`, `.send`, and interface calls. For each: what can the callee do? What state is exposed?

15. **`delegatecall` correctness** — storage layout alignment between proxy and implementation. Context preservation. Check `variable-order` printer for both contracts.

16. **Return value handling** — unchecked low-level calls, `try`/`catch` that silently swallows errors, missing success check on `.call`.

17. **Non-standard token behavior** — if the function handles arbitrary ERC20 tokens, check against `resources/checklists/non-standard-tokens.md` (fee-on-transfer, rebasing, missing return values, pausable, low decimals, callbacks).

## Boundary Conditions

18. **Boundary values** — zero amounts, `type(uint256).max`, empty arrays, first depositor / empty vault attacks. What happens at the extremes?

19. **Timestamp dependence** — `block.timestamp` in critical logic? Validators can manipulate timestamps by ~15 seconds.

20. **ERC compliance** — return values, zero/self-transfer edge cases, event emissions per the standard.

## Signatures & Replay

21. **Signature malleability** — `ecrecover` without `s`-value normalization (use OpenZeppelin `ECDSA`), missing `address(0)` check on recovered signer, cross-chain replay (missing chain ID in domain separator).

22. **Permit front-running** — attacker front-runs a `permit` + `transferFrom` combo by calling `permit` with the victim's signature first. Victim's tx reverts. Safe pattern: wrap `permit` in try/catch.

## Composability & Multi-Step

23. **Multi-call / batch atomicity** — can combining multiple operations in one transaction break invariants that hold for individual calls?

24. **Event emission correctness** — wrong parameter values in events, missing events for critical state changes (ownership transfer, parameter updates), events emitted before state finalized.

## EVM-Specific

25. **Transient storage (EIP-1153)** — if `TSTORE`/`TLOAD` used: reentrancy guards relying on transient storage clear at end of transaction. Nested calls within the same tx share transient storage. A callback from an external contract may bypass the guard if using a different slot.
