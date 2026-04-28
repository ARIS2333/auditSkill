# Other Domain-Specific Playbooks

This file contains playbooks for less-common protocol types. Each section is a standalone checklist.

---

## Cross-Chain / Bridge

- **Message replay:** Can a valid cross-chain message be replayed on the same or different chain? Check for nonce/hash tracking.
- **Message ordering:** Does the protocol assume messages arrive in order? Bridges generally do NOT guarantee ordering.
- **Finality assumptions:** Does the protocol wait for sufficient confirmations? Reorgs on the source chain can invalidate messages.
- **Relayer trust:** Can the relayer censor, reorder, or delay messages? What happens if the relayer is down?
- **Gas limit on destination:** If the destination-chain execution runs out of gas, is the message lost or retryable?
- **Token mapping:** Are bridged token representations correctly mapped? Can an attacker mint on one side without a corresponding lock on the other?

---

## Staking / Restaking

- **Withdrawal delay manipulation:** Can an attacker trigger a withdrawal delay and then exploit the locked-in position (e.g., slashing while funds are in cooldown)?
- **Slashing accounting:** Does slashing reduce stake proportionally? Can slashing cause underflow or leave dust?
- **Reward distribution:** Is reward calculation based on time-weighted stake? Can a just-in-time staker capture rewards disproportionately?
- **Operator delegation:** If users delegate to operators, can a malicious operator grief delegators (intentional slashing, withholding rewards)?
- **Unbonding queue:** Is there a maximum queue length? Can the queue be DoS'd by many small unbonding requests?
- **Re-staking compounding:** If rewards are automatically restaked, is the compounding math correct? Does it account for rounding?
- **Checkpoint/epoch boundaries:** Can operations that span epoch boundaries produce inconsistent state?
- **Minimum stake:** Can dust stakes pollute the staker set or consume unbounded storage?

---

## Governance / Timelock

- **Flash loan governance:** Can an attacker flash-mint or flash-borrow governance tokens, vote, and return them within one transaction? Check if voting power is snapshot-based or live-balance-based.
- **Proposal front-running:** Can an attacker see a pending proposal and front-run it (e.g., moving funds before a parameter change takes effect)?
- **Timelock bypass:** Are there emergency functions that skip the timelock? Who controls them?
- **Quorum manipulation:** Can the quorum threshold be changed to a very low value by the current governance?
- **Proposal cancellation:** Who can cancel proposals? Can a malicious actor cancel legitimate proposals?
- **Execute timing:** Is there a grace period after the timelock delay? What happens if execution is delayed beyond it?

---

## Perpetuals / Derivatives

- **Funding rate manipulation:** Can a large position holder skew the funding rate to extract value from counterparties?
- **Mark price vs index price divergence:** If the mark price deviates significantly from the index, can positions be unfairly liquidated?
- **Liquidation cascade:** Can liquidating one large position crash the mark price and trigger cascading liquidations?
- **ADL (Auto-Deleveraging):** If the insurance fund is depleted, is the ADL mechanism fair? Can it be gamed?
- **Position sizing limits:** Are there maximum position sizes? Can an attacker open a position larger than the protocol can handle?
- **Oracle latency:** Derivatives are extremely sensitive to oracle timing — a few seconds of staleness can cause incorrect liquidations.
- **Margin calculation:** Is isolated vs cross-margin correctly implemented? Can switching margin mode create an exploitable window?
- **Fee-on-close vs fee-on-open:** Are fees applied correctly and consistently? Can a zero-profit round-trip result in a loss greater than fees?
- **Negative PnL overflow:** Can a position's loss exceed the deposited margin? Who absorbs the excess?

---

## Token Launch / Bonding Curve

- **Bonding curve manipulation:** Can early buyers extract disproportionate value from later buyers by timing purchases around curve inflection points?
- **LBP (Liquidity Bootstrapping Pool) sniping:** Can MEV bots snipe the launch by buying in the first block?
- **Vesting cliff bypass:** Can tokens locked under vesting be transferred, delegated, or used as collateral before the cliff?
- **Unlock schedule math:** Is the linear unlock calculation correct? Off-by-one in time boundaries can unlock tokens early or late.
- **Rug pull vectors:** Can the token creator drain liquidity, pause transfers, or mint unlimited tokens after launch?
- **Maximum supply enforcement:** Is there a hard cap that cannot be bypassed by the owner or governance?

---

## Yield Aggregator (Multi-Strategy)

- **Strategy trust:** When the vault deposits into a strategy, can a malicious strategy steal or lock funds? Is there a maximum allocation limit per strategy?
- **Harvest sandwich:** Can a bot sandwich the `harvest()` transaction to extract value from yield being compounded?
- **Strategy withdrawal ordering:** If the vault needs to withdraw but the primary strategy has locked funds, does it try secondary strategies? Can this ordering be exploited?
- **Emergency withdrawal:** Does the emergency path sacrifice yield but guarantee fund recovery? Can it be called by anyone, or only admin?
- **Total assets calculation across strategies:** Does `totalAssets` correctly sum assets in all strategies, accounting for unrealized gains/losses?
- **Strategy migration:** When rotating from old to new strategy, is there a window where funds are in transit and neither strategy accounts for them?
- **Reward token handling:** Are reward tokens (CRV, COMP, etc.) correctly swapped and compounded? Can the swap be sandwiched?
