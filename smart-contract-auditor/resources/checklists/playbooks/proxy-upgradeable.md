# Proxy / Upgradeable Playbook

## 1. Uninitialized Implementation

**What it is:** The implementation contract's `initialize()` is never called on the implementation itself (only on the proxy). An attacker calls `initialize()` on the bare implementation, takes ownership, then calls `selfdestruct` (pre-Dencun) or `upgradeToAndCall` with a malicious implementation.

**What to look for:**
- Missing `_disableInitializers()` in the implementation's constructor
- `initialize()` callable without the `initializer` modifier
- Implementation contract deployed without constructor protection

**Vulnerable pattern:**
```solidity
contract VaultImpl is UUPSUpgradeable {
    function initialize(address owner) public initializer {
        __Ownable_init(owner);
    }
    // Missing: constructor() { _disableInitializers(); }
}
```

**Safe pattern:**
```solidity
constructor() {
    _disableInitializers();
}
```

**PoC strategy:** Use `vm.prank` as an attacker, call `initialize()` on the implementation address (not the proxy), assert ownership was taken.

---

## 2. Storage Layout Collision

**What it is:** The proxy and implementation share storage via `delegatecall`. If the implementation's storage layout doesn't match the proxy's expectations, variables can overlap and corrupt each other.

**What to look for:**
- State variables added in the wrong order between upgrade versions
- Missing `__gap` arrays in base contracts (prevents future base contract additions from shifting derived storage)
- ERC-7201 namespaced storage not used where it should be
- Inherited contracts that declare state variables (each one shifts the derived contract's slots)

**Verification:** Run `slither . --print variable-order` on both old and new implementations. Compare slot-by-slot.

**PoC strategy:** Deploy V1 with known state, upgrade to V2, read the same slot and show the value has been corrupted or reinterpreted.

---

## 3. Function Selector Collision

**What it is:** The proxy contract may have its own functions (e.g., `admin()`, `upgradeTo()`). If the implementation has a function with the same 4-byte selector, calls intended for the proxy hit the implementation instead (or vice versa).

**What to look for:**
- Run `slither . --print function-id` on both proxy and implementation
- Compare selectors — any collision is a critical finding
- Transparent proxy pattern is designed to prevent this (admin calls go to proxy, user calls go to implementation), but only if the proxy correctly routes based on `msg.sender == admin`

---

## 4. Upgrade Authorization

**What to look for:**
- **UUPS:** `_authorizeUpgrade()` must have proper access control. If it's empty or unguarded, anyone can upgrade.
- **Transparent:** Only the ProxyAdmin contract should be able to call `upgradeTo()`. Is the ProxyAdmin properly secured?
- **Diamond:** `diamondCut()` must be restricted. Check for facet replacement that could swap critical functions.
- Is there a timelock between proposing and executing an upgrade?
- Can the upgrade mechanism be used for a rug pull (upgrade to a malicious implementation that drains funds)?

---

## 5. Initializer Re-Entrancy and Re-Initialization

**What to look for:**
- Can `initialize()` be called again after the first call? (Missing `initializer` modifier, or `reinitializer` used incorrectly)
- Multiple `initialize` functions (e.g., one per inherited contract) — are they all called in the correct order?
- Parent `initialize` calls missing: if `__Ownable_init()` is never called, ownership is unset.
- `reinitializer(N)` version number — is it correctly incremented for each upgrade that needs re-initialization?

---

## 6. `delegatecall` Context Hazards

**What to look for:**
- `msg.sender` and `msg.value` are preserved across `delegatecall` — the implementation sees the original caller. This is correct for proxies but dangerous if the implementation itself makes `delegatecall` to untrusted targets.
- `address(this)` in the implementation refers to the proxy's address, not the implementation's. Any logic that uses `address(this)` for self-reference will behave differently when called directly vs. via proxy.
- `immutable` variables in the implementation are stored in bytecode, not storage — they retain the implementation's deployment values, not the proxy's context.

---

## 7. Upgrade-Specific State Migration

**What to look for:**
- Does the upgrade require data migration (e.g., changing the encoding of a mapping, splitting a variable into two)?
- Is the migration performed atomically with the upgrade (via `upgradeToAndCall`)?
- If migration is a separate transaction, is there a window where the contract is in an inconsistent state?

---

## 8. Proxy-Specific Slither Checks

Run `slither-check-upgradeability` for automated detection. See `resources/tools/slither.md` §Upgradeability Analysis for full command syntax and the 17 checks performed.
