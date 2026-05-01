# Privilege & Management Risk Checklist

Reference for analyzing centralization risks, key compromise scenarios, and privilege escalation paths. Use during Phase 3 (Attack Planning) to assess what damage a compromised or malicious privileged role could inflict.

**When to use:** During Phase 3, after reviewing the Phase 2 Access Control Matrix and Value Flow sections. For every privileged role identified in the codebase, work through this checklist.

---

## Step 1: Enumerate All Privileged Roles

From the Phase 1 structural summary (Entry Points & Access Control table) and Phase 2 codebase overview (Access Control Matrix), list every distinct role:

| Role | Address Type | How Set | Can Be Changed By |
|------|-------------|---------|-------------------|
| [owner] | [EOA / Multisig / Timelock / Governance] | [constructor / initialize] | [self / governance vote] |

Common roles to look for: `owner`, `admin`, `operator`, `guardian`, `pauser`, `minter`, `upgrader`, `keeper`, `relayer`, `treasury`, `fee setter`.

---

## Step 2: Per-Role Power Analysis

For each role, enumerate what it can do:

### 2.1 Direct Fund Access
- Can this role withdraw/transfer funds from the protocol?
- Can this role set fee recipients to its own address?
- Can this role set fees to 100%?
- Can this role mint tokens without limit?
- Can this role redirect yield/rewards?

### 2.2 Indirect Fund Access
- Can this role upgrade the implementation contract (and thus gain arbitrary control)?
- Can this role change oracle addresses (and thus manipulate prices)?
- Can this role change whitelisted tokens/strategies (and thus introduce malicious contracts)?
- Can this role change withdrawal addresses or beneficiaries?
- Can this role modify accounting parameters to inflate/deflate values?

### 2.3 Denial of Service
- Can this role pause the protocol with no unpause path?
- Can this role freeze user funds indefinitely?
- Can this role block withdrawals/liquidations?
- Can this role set parameters to values that brick the protocol (zero divisors, overflow-inducing values)?

### 2.4 Role Escalation
- Can this role grant itself additional roles?
- Can this role grant roles to arbitrary addresses?
- Can this role remove other privileged roles?
- Can this role change the governance/timelock parameters?
- Can this role bypass timelock delays?

---

## Step 3: Key Compromise Scenario Analysis

For each role, assess the worst case if the private key is compromised:

| Role | Worst-Case Scenario | Funds at Risk | Time to Exploit | Recovery Path |
|------|-------------------|---------------|-----------------|---------------|
| [owner] | [e.g., upgrade to malicious impl, drain all funds] | [e.g., 100% TVL] | [e.g., instant / after timelock] | [e.g., none / guardian can pause] |

### Key Questions
- **Single point of failure?** Is this role a single EOA? A multisig? How many signers?
- **Timelock protection?** Is there a delay between proposing and executing privileged actions? How long?
- **Guardian/emergency override?** Can another role pause or block the compromised role's actions?
- **Revocability?** Can the compromised role be removed/replaced without the compromised key?

---

## Step 4: Mitigation Assessment

For each critical privilege risk, check whether mitigations exist:

| Mitigation | Present? | Details |
|-----------|----------|---------|
| Timelock on admin actions | [Yes/No] | [Duration, which actions] |
| Multisig requirement | [Yes/No] | [Threshold, total signers] |
| Maximum value caps | [Yes/No] | [e.g., max fee = 10%, max withdrawal per tx] |
| Pausability (separate role) | [Yes/No] | [Who can pause, who can unpause] |
| Two-step ownership transfer | [Yes/No] | [Pending owner must accept] |
| Role separation | [Yes/No] | [Different keys for different functions] |
| On-chain governance | [Yes/No] | [Voting period, quorum] |

---

## Step 5: Output Format

Document findings in the Phase 3 attack plan under the Privilege & Management Risk section (section 4 of the attack plan template). Critical findings (single EOA can drain funds with no timelock) should be flagged for Phase 4 write-up as Centralization findings (C-XX).
