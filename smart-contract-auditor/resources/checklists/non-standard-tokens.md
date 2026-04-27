# Non-Standard Token Behavior Checklist

Reference for auditing protocols that interact with arbitrary ERC20 tokens. Many DeFi exploits originate from assumptions about "normal" token behavior that specific tokens violate.

**When to use:** During Phase 4 code reading, whenever the protocol accepts external tokens (LP vaults, DEX routers, lending pools, bridges, yield aggregators). Cross-reference every `transfer`, `transferFrom`, `approve`, `balanceOf`, and balance-based accounting pattern against this list.

---

## Token Behavior Matrix

### 1. Fee-on-Transfer (Deflationary) Tokens

**Examples:** STA, PAXG (0.02% fee), certain SafeMoon forks

**Behavior:** The recipient receives fewer tokens than the sender sends. `balanceOf(recipient)` increases by less than `amount` passed to `transfer(recipient, amount)`.

**How to detect in code:**
- Any pattern that assumes `balanceOf(this)` increased by exactly `amount` after `transferFrom`
- Missing balance-before/balance-after accounting pattern
- Share calculation using `amount` instead of actual received amount

**Vulnerable pattern:**
```solidity
// BAD: assumes full amount received
token.transferFrom(msg.sender, address(this), amount);
shares = amount * totalShares / totalAssets; // amount is wrong
```

**Safe pattern:**
```solidity
uint256 balBefore = token.balanceOf(address(this));
token.transferFrom(msg.sender, address(this), amount);
uint256 received = token.balanceOf(address(this)) - balBefore;
shares = received * totalShares / totalAssets;
```

**PoC strategy:** Deploy a mock token with a 5% transfer fee, deposit via the vulnerable function, assert that internal accounting diverges from actual balance.

---

### 2. Rebasing Tokens

**Examples:** stETH (Lido), AMPL (Ampleforth), aTokens (Aave)

**Behavior:** Token balances change automatically without transfers. Can be positive rebase (balance increases) or negative rebase (balance decreases).

**How to detect in code:**
- Protocol caches `balanceOf` and uses it later (stale after rebase)
- Internal accounting tracks a fixed amount but actual balance has changed
- Withdrawals based on cached amounts (may exceed actual balance after negative rebase)

**Key risks:**
- **Positive rebase:** Excess tokens trapped in contract with no way to withdraw (locked yield)
- **Negative rebase:** Protocol becomes insolvent — promises more than it holds
- **Share-based wrappers (wstETH)** solve this — check if the protocol uses the rebasing or wrapped version

**PoC strategy:** Fork mainnet, deposit rebasing tokens, simulate a rebase (use `vm.store` to manipulate total supply or shares), attempt withdrawal, assert insolvency or trapped funds.

---

### 3. Tokens with Missing Return Values

**Examples:** USDT (mainnet), BNB (BEP20 on BSC), OMG

**Behavior:** `transfer()` and `approve()` do not return a `bool`. Solidity's ABI decoder expects return data — calling these via a standard `IERC20` interface reverts on decode.

**How to detect in code:**
- Direct `IERC20(token).transfer()` or `IERC20(token).approve()` without `SafeERC20`
- Absence of `safeTransfer` / `safeApprove` wrappers
- Custom transfer wrappers that don't handle zero-length return data

**Safe pattern:** Use OpenZeppelin's `SafeERC20` or Solmate's `SafeTransferLib`, which handle both returning and non-returning tokens.

**Note:** USDT's `approve()` also requires setting allowance to 0 before setting a new non-zero value (see #7 below).

---

### 4. Pausable Tokens

**Examples:** USDC, USDT, WBTC (some implementations)

**Behavior:** A privileged address can pause all transfers. While paused, `transfer`, `transferFrom`, `approve`, `mint`, and `burn` revert.

**How to detect in code:**
- Withdrawal functions that require a token transfer as part of the execution path
- Liquidation functions that must move the paused token
- Any critical protocol operation that becomes blocked if the token is paused

**Key risks:**
- **DoS on withdrawals:** Users cannot exit positions
- **Blocked liquidations:** Positions become unliquidatable, protocol accrues bad debt
- **Stuck protocol state:** State transitions that depend on successful token transfer

---

### 5. Blacklistable / Blocklist Tokens

**Examples:** USDC, USDT, BUSD

**Behavior:** Token admin can blacklist specific addresses. Transfers to/from blacklisted addresses revert.

**How to detect in code:**
- Protocol holds tokens on behalf of users — if the protocol address is blacklisted, all users lose funds
- Pull-over-push patterns where the recipient address might be blacklisted
- Withdrawal functions that send tokens to `msg.sender` — if user gets blacklisted, their funds are stuck

**Key risks:**
- Protocol contract address blacklisted → catastrophic fund lock
- Individual user blacklisted → their share of funds locked in protocol
- Liquidation target blacklisted → can't liquidate, bad debt accrues

**PoC strategy:** Mock the blacklist function, blacklist the protocol or a user, attempt operations, assert they revert.

---

### 6. Low-Decimal Tokens

**Examples:** USDC (6), USDT (6), WBTC (8), GUSD (2)

**Behavior:** Fewer decimal places means larger minimum units. Math operations that work fine with 18-decimal tokens can have catastrophic precision loss with 6-decimal tokens.

**How to detect in code:**
- Hardcoded `1e18` denominators in share/price calculations
- Division that produces zero for small-but-valid amounts
- Minimum deposit/amount checks calibrated for 18 decimals
- First-depositor inflation attacks are much cheaper with low-decimal tokens

**Specific danger:** In share-based vaults, the "donation attack" (ERC4626 inflation) becomes viable with as little as $1 of USDC but would require millions with a 18-decimal token.

---

### 7. Approval Race Condition Tokens

**Examples:** USDT (mainnet), any token without `increaseAllowance`/`decreaseAllowance`

**Behavior:** USDT's `approve()` reverts if current allowance is non-zero and new value is non-zero. Must set to 0 first, then set new value.

**How to detect in code:**
- `token.approve(spender, newAmount)` without first resetting to 0
- Missing `safeApprove` or `forceApprove` pattern

**Safe patterns:**
```solidity
// OpenZeppelin SafeERC20.forceApprove (recommended)
token.forceApprove(spender, amount);

// Manual pattern
token.approve(spender, 0);
token.approve(spender, amount);
```

---

### 8. Tokens with Callbacks (Hooks)

**Examples:** ERC777 (`tokensReceived`), ERC1155 (`onERC1155Received`), ERC721 (`onERC721Received`)

**Behavior:** Token transfers invoke callback functions on the sender or receiver. This creates reentrancy vectors even when the protocol doesn't make explicit external calls.

**How to detect in code:**
- Token transfers before state updates (classic reentrancy, but triggered by the token itself)
- Missing reentrancy guards on functions that interact with hook-capable tokens
- `safeTransferFrom` on ERC721/ERC1155 (the "safe" prefix means it calls hooks — ironically making it more dangerous)

**Key insight:** Slither's `reentrancy-eth` detector often misses these because the external call is hidden inside the token's `transfer` implementation, not visible in the audited contract's code.

---

### 9. Upgradeable / Proxy Tokens

**Examples:** USDC (upgradeable proxy), TUSD, many newer tokens

**Behavior:** Token logic can be changed by the admin at any time. A token that behaves "normally" today could add fees, blacklists, or entirely new logic tomorrow.

**How to detect in code:**
- Trust assumptions about token behavior that may not hold after an upgrade
- Hardcoded behavior expectations for upgradeable tokens
- Long-lock or time-delayed positions denominated in upgradeable tokens

**Audit relevance:** This is primarily a **trust assumption** to document (Phase 2, Section 13: Known Risks), not a code bug to fix.

---

### 10. Tokens with Multiple Entry Points

**Examples:** TUSD (old implementation had two contracts that both updated the same balance)

**Behavior:** Two different contract addresses can move the same underlying balance. Protocols that whitelist token addresses may only know about one entry point.

**How to detect in code:**
- Token whitelist / allowlist checks that can be bypassed via the secondary entry point
- Balance checks on one address that don't account for operations via the other

---

### 11. Tokens that Revert on Zero-Amount Transfers

**Examples:** LEND (Aave V1 token), some custom tokens

**Behavior:** `transfer(to, 0)` or `transferFrom(from, to, 0)` reverts instead of being a no-op.

**How to detect in code:**
- Withdrawal/claim functions where the amount could legitimately be zero
- Loop-based distributions where some recipients get zero
- Fee collection where the fee rounds down to zero

**Safe pattern:** Guard with `if (amount > 0)` before transfer calls.

---

### 12. Tokens with Transfer Limits / Maximum Balances

**Examples:** Some reflection tokens, anti-whale tokens

**Behavior:** Transfers above a certain threshold or to addresses holding above a maximum revert.

**How to detect in code:**
- Protocols that aggregate user deposits into a single address (vault pattern) — may hit max balance
- Large withdrawals that exceed per-tx limits

---

### 13. Tokens with Permit (EIP-2612)

**Examples:** DAI, USDC (V2+), most modern tokens

**Behavior:** Gasless approvals via signed message (`permit`). The `permit` function can be called by anyone with a valid signature.

**How to detect in code:**
- `permit` + `transferFrom` in the same function — front-running: attacker calls `permit` first, victim's tx reverts on duplicate permit
- Missing `try/catch` around `permit` calls
- Signature replay across chains (missing `DOMAIN_SEPARATOR` chain ID check)

**Specific attack:**
```
1. Alice signs permit for Protocol
2. Bob sees mempool, front-runs: calls permit(Alice's sig)
3. Alice's tx calls permit(same sig) -> reverts (nonce already used)
4. Alice's deposit fails, but her allowance is now set
```

**Safe pattern:** Wrap `permit` in try/catch — if it fails, check if allowance is already sufficient.

---

### 14. Flash-Mintable Tokens

**Examples:** DAI (`flash`), any token implementing ERC-3156

**Behavior:** Attacker can mint unlimited tokens temporarily within a single transaction.

**How to detect in code:**
- Governance voting weighted by token balance (flash-mint → vote → repay)
- Price oracles using spot balance or reserves
- Any check that assumes token supply or balance can't change drastically within one block

---

## Quick Reference Table

| Behavior | Tokens | Primary Risk | Detection Pattern |
|----------|--------|-------------|-------------------|
| Fee-on-transfer | STA, PAXG | Accounting mismatch | No balance-diff check after transfer |
| Rebasing | stETH, AMPL, aTokens | Insolvency / locked funds | Cached `balanceOf` used across time |
| Missing return value | USDT, BNB, OMG | Revert on transfer | No `SafeERC20` / no returndata handling |
| Pausable | USDC, USDT | DoS | Critical path requires transfer |
| Blacklistable | USDC, USDT | Fund lock | Protocol holds funds for users |
| Low decimals | USDC (6), WBTC (8), GUSD (2) | Precision loss / cheap inflation attack | Hardcoded 1e18, division producing zero |
| Approval race | USDT | Revert on approve | No `forceApprove`, no reset-to-zero |
| Token callbacks | ERC777, ERC1155, ERC721 | Hidden reentrancy | Transfer before state update |
| Upgradeable | USDC, TUSD | Trust assumption | Logic may change after audit |
| Multiple entry points | TUSD (old) | Whitelist bypass | Single-address allowlist |
| Zero-amount revert | LEND | DoS on zero-amount paths | No `amount > 0` guard |
| Transfer limits | Anti-whale tokens | DoS on aggregation | Single address aggregating deposits |
| Permit front-running | DAI, USDC V2+ | Tx revert griefing | `permit` without try/catch |
| Flash-mintable | DAI, ERC-3156 | Governance / price manipulation | Balance-weighted decisions |

---

## Audit Integration

During **Phase 0.4** (Detect Audit Scenarios), if the protocol accepts arbitrary ERC20 tokens or interacts with specific tokens from this list:
1. Flag the token behavior categories that apply
2. During **Phase 4** code reading, check every token interaction against the relevant categories above
3. For each match, classify as confirmed/potential per the Phase 4.6 triage framework
4. Document token assumptions in the Phase 2 codebase overview (Section 13: Known Risks)
