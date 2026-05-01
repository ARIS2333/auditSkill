# Governance / Timelock Playbook

## 1. Flash Loan Governance

**What it is:** Attacker flash-mints or flash-borrows governance tokens, votes, and returns them within one transaction.

**What to look for:**
- Voting power based on live balance vs snapshot — live balance is flash-loan-manipulable
- Missing snapshot mechanism (ERC20Votes `getPastVotes`)
- Proposal creation threshold checked against live balance

**PoC strategy:** Flash-borrow governance tokens, create or vote on a proposal, return tokens. Assert the vote was recorded.

---

## 2. Proposal Front-Running

**What to look for:**
- Can an attacker see a pending proposal and front-run it (e.g., moving funds before a parameter change takes effect)?
- Is proposal content visible in the mempool before execution?
- Can users exit positions during the timelock delay to avoid unfavorable changes?

---

## 3. Timelock Bypass

**What to look for:**
- Are there emergency functions that skip the timelock? Who controls them?
- Can the timelock delay be set to zero?
- Can a proposer cancel and re-propose to reset the delay?
- Is the timelock enforced at the contract level or only by convention?

---

## 4. Quorum Manipulation

**What to look for:**
- Can the quorum threshold be changed to a very low value by the current governance?
- Is quorum checked at proposal creation, voting end, or execution?
- Can abstentions or delegated-then-undelegated votes affect quorum calculation?

---

## 5. Proposal Cancellation

**What to look for:**
- Who can cancel proposals? Can a malicious actor cancel legitimate proposals?
- Is there a cancellation threshold or penalty?
- Can the proposer cancel after voting has started to prevent unfavorable outcomes?

---

## 6. Execute Timing

**What to look for:**
- Is there a grace period after the timelock delay? What happens if execution is delayed beyond it?
- Can execution be front-run by other transactions that change the expected state?
- Can the same proposal be executed multiple times?
