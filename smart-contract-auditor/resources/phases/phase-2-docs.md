# Phase 2: Codebase Documentation

**Purpose:** Dive into the source code using Phase 1's structural model as your guide. Produce comprehensive codebase documentation that becomes the shared reference for all subsequent phases. This is where you build the mental model that drives scanning, code reading, PoC development, and reporting.

**Gate:** Document covers all 13 sections from the template. Fact-Checking Checklist passes completely. No unverified claims. Ranked hypothesis list produced.

**Inputs:**
- `audit-output/phase-1-recon/structural-summary.md` — your primary structural reference
- `audit-output/phase-1-recon/preliminary-hypotheses.md` — structural-only hypothesis list to refine
- `audit-output/phase-1-recon/coverage-report.txt` (if tests exist)
- `audit-output/phase-1-recon/printers/` — raw printer files (fallback for details not in structural summary)
- `.sol` files in scope

**Outputs:**
- `audit-output/phase-2-docs/codebase-overview.md` — comprehensive codebase documentation
- `audit-output/phase-2-docs/hypothesis-list.md` — ranked attack hypothesis list

**Checkpoint:** Present the codebase overview and refined hypothesis list to the user. Pause and wait for confirmation before proceeding to Phase 3.

---

## Step 1: Read the Methodology and Template

Read `resources/templates/codebase-report-guide.md` for the population methodology — which structural summary sections and printer files feed each template section.

Read `resources/templates/codebase-report.md` for the 13-section template to fill in.

---

## Step 2: Determine Reading Order

Before diving into code, establish which contracts to read and in what order.

1. **Start from the inheritance graph** in the structural summary (section 2). Read base contracts before derived contracts — you need to understand modifier behavior, storage layout, and internal functions before reading the contracts that inherit them.

2. **Within that order, prioritize by the Phase 1 hypothesis ranking.** Contracts containing Critical/High-priority targets from `preliminary-hypotheses.md` get read first.

3. **For large codebases (>15 contracts):** Do NOT attempt to read everything in one pass. Batch contracts into groups of 3-5 related contracts (e.g., "core vault + strategy + router", "governance + timelock"). Complete the template sections for each batch before moving to the next. Write intermediate results to disk.

---

## Step 3: Read Code and Populate the Template

For each contract, the goal at this stage is **understanding intent, not finding bugs.** You are building the map that Phase 4 will use to find bugs. Focus on:

- **What does this contract do?** (purpose, role in the system)
- **What state does it manage?** (storage variables, their relationships)
- **Who can call it and what happens?** (access control, state transitions, value flows)
- **What external dependencies does it have?** (other contracts, oracles, tokens)
- **What must always be true?** (invariants, both explicit and implicit)

### Per-Contract Reading Procedure

For each contract in your reading order:

1. **Open the structural summary** — read the contract's entry in section 3 (Entry Points & Access Control table). Note all external/public functions, their modifiers, and state variables written. This gives you a verified skeleton.

2. **Read the contract source code.** For each function:
   - Verify the structural summary data matches what you see in code. If they disagree, the printer is correct — re-read.
   - Understand the function's purpose and how it fits into user flows.
   - Note any inline access control (`require(msg.sender == ...)`) that modifiers don't capture.
   - Trace the complete state change: what gets read, what gets written, what events fire.

3. **Populate the relevant template sections** for this contract:
   - Add it to the Contract Inventory (section 4)
   - Write its Contract Deep Dive (section 5)
   - Add its functions to User Flows (section 6) and Access Control Matrix (section 7) as appropriate
   - Trace how value moves through it for the Value Flow section (section 8)

4. **After each contract, write your progress to disk.** Do not hold the entire codebase model in memory.

### Context Management

- **Read the structural summary first for each section**, not raw printer files. The structural summary already contains the inheritance tree, function-modifier-state-write table, unguarded functions, and storage layout.
- **Only fall back to raw printer files** (via the printer output index in the structural summary section 6) when you need detail the summary does not cover — e.g., DOT graph traversal for call paths, per-function `require.txt` conditions, full `not-pausable.txt` list.
- **After writing each section, subsequent sections can reference what you already wrote.** The cycle is: structural summary → code reading → raw printer if needed → write section → move on.

---

## Step 4: Complete System-Level Sections

After reading all contracts individually, populate the cross-cutting sections that require understanding the whole system:

- **System Overview** (section 1) — write this last or near-last, since it requires holistic understanding
- **Architecture Diagram** (section 3) — draw the contract relationships from the inheritance tree + call graph
- **State Machine** (section 9) — only if discrete states exist (paused/active/emergency)
- **Key Invariants** (section 12) — synthesize from code reading, both explicit invariants and implicit ones
- **Known Risks & Trust Assumptions** (section 13) — document every external trust dependency

---

## Step 5: Run Fact-Checking Checklist

Run through the Fact-Checking Checklist at the bottom of `resources/templates/codebase-report.md`. Every checkbox must pass before the document is complete.

**Structural verification:** Cross-check every claim against the structural summary and printer output.

**Flow verification:** At least one happy-path user flow must be verified by tracing through code or running `forge test -vvvv`.

**Uncertainty handling:** Flag any unverified assumptions explicitly.

---

## Step 6: Refine Attack Hypothesis List

You now have both the structural model (Phase 1 printers) and deep protocol understanding (code reading). Start from `audit-output/phase-1-recon/preliminary-hypotheses.md` and produce a refined list.

Write to **`audit-output/phase-2-docs/hypothesis-list.md`**.

### Re-rank and expand the list:

- Confirm or deprioritize each Phase 1 hypothesis based on code-level understanding.
- Add new hypotheses discovered during code reading.
- For each hypothesis, note which template sections provide supporting evidence.

### Priority criteria:

| Priority | Criteria |
|----------|----------|
| **Critical** | State-modifying, no modifiers, no require statements |
| **High** | Handles ETH/token transfers with access control (verify guard correctness) |
| **Medium** | Complex functions with many state writes and cross-function calls |
| **Low** | View functions, well-guarded administrative functions |
| **Deprioritized** | Structural signal appears benign after code reading — kept for Phase 4 re-validation |

### Never Permanently Dismiss Hypotheses

**Do NOT mark any hypothesis as "Resolved", "Invalid", or "False Positive" in Phase 2.** Phase 2 only has code-reading-level understanding — it lacks Phase 4's adversarial thinking framework, domain playbooks, cross-function pattern analysis, and impact amplification analysis.

Mark seemingly-benign hypotheses as **Deprioritized** with your reasoning. They remain in the list and are mandatory re-validation targets in Phase 4. Format:

```markdown
| # | Target | Signal | Priority | Phase 2 Notes |
|---|--------|--------|----------|---------------|
| 24 | Registry.unregisterVault() | try/catch + balanceOf check | Deprioritized | Appears intentional safety — revisit in Phase 4 with donation/griefing lens |
```

Compare the refined list against the Phase 1 preliminary list: note which structural signals were confirmed, which were deprioritized (with justification), and what new targets emerged from code reading.
