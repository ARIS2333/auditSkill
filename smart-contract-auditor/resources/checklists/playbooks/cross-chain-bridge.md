# Cross-Chain / Bridge Playbook

## 1. Message Replay

**What it is:** A valid cross-chain message is replayed on the same or different chain, causing duplicate execution.

**What to look for:**
- Missing nonce or hash tracking for processed messages
- Nonce tracked per-chain but not per-sender or per-destination
- Hash computed without chain ID, allowing cross-chain replay

---

## 2. Message Ordering

**What to look for:**
- Protocol logic that assumes messages arrive in order — bridges generally do NOT guarantee ordering
- Dependent operations (e.g., "create account" must arrive before "deposit") that break if reordered
- Missing sequence numbers or out-of-order handling

---

## 3. Finality Assumptions

**What to look for:**
- Protocol acts on messages before sufficient confirmations on the source chain
- Reorgs on the source chain can invalidate messages already processed on the destination
- Different finality guarantees across chains (Ethereum ~12 min, L2s vary)

---

## 4. Relayer Trust

**What to look for:**
- Can the relayer censor, reorder, or delay messages?
- What happens if the relayer is down — are messages lost or queued?
- Can users self-relay as a fallback?
- Does the relayer have any privileged actions beyond message delivery?

---

## 5. Gas Limit on Destination

**What to look for:**
- If the destination-chain execution runs out of gas, is the message lost or retryable?
- Hardcoded gas limits that may be insufficient for complex operations
- Gas price spikes on the destination chain blocking execution

---

## 6. Token Mapping

**What to look for:**
- Are bridged token representations correctly mapped between chains?
- Can an attacker mint on one side without a corresponding lock on the other?
- What happens if the canonical token on one chain is upgraded or paused?
- Double-spending via race conditions between lock and mint
