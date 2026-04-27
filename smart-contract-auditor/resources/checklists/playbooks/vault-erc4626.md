# Vault / ERC4626 Playbook

- First depositor inflation attack (donate to inflate share price, victim gets 0 shares)
- Share price manipulation via direct token transfer (donation attack)
- Rounding direction: deposits should round DOWN (fewer shares), withdrawals should round DOWN (fewer assets) — always favor the vault
- `deposit` vs `mint` vs `redeem` vs `withdraw` — all four must be consistent
- Empty vault edge case (totalSupply == 0)
