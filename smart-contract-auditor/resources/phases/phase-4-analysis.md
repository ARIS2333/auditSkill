# Phase 4: Targeted Code Reading & Triage

**Purpose:** Deep security-focused code reading guided by Phases 1-3. This is the core vulnerability discovery phase. Classify findings as you go.

**Gate:** All critical and high-target functions read. Every High and Medium finding (automated and manual) classified with written justification. All Phase 2 Deprioritized hypotheses re-validated.

**Inputs:**
- `audit-output/phase-1-recon/structural-summary.md` (structural reference)
- `audit-output/phase-2-docs/hypothesis-list.md` (ranked targets)
- `audit-output/phase-2-docs/codebase-overview.md` (your map)
- `audit-output/phase-3-scanning/scan-summary.md` (detector findings)

**Outputs:**
- `audit-output/phase-4-analysis/code-reading-notes.md` — observations per function, cross-referenced against structural summary
- `audit-output/phase-4-analysis/triage.md` — every finding classified with justification

**Reference:** Keep `audit-output/phase-2-docs/codebase-overview.md` open as your map. The architecture diagram tells you where to look, the user flows tell you what to trace, the access control matrix tells you what to verify, the key invariants tell you what to try to break.

---

## Step 1: Determine Reading Order

From the inheritance graph in the structural summary, determine dependency order: **read base contracts before derived contracts.** Within that order, prioritize by Phase 2 hypothesis ranking, weighted by Phase 3 detector findings.

For base contracts: verify that modifier behavior matches the name — a modifier called `onlyOwner` that doesn't check ownership is a critical finding.

---

## Step 2: Apply the Code Reading Checklist

Read `resources/checklists/code-reading-checklist.md`. Apply every item to each function flagged in the hypothesis list and each function flagged by Phase 3 detectors.

For each function:
1. **Verify Phase 1 data** — does the function actually write the variables listed in `function-summary`?
2. **Work through the checklist systematically** — don't skip items.
3. **Record observations** in `audit-output/phase-4-analysis/code-reading-notes.md` as you go.

---

## Step 3: Apply Adversarial Thinking

Read `resources/checklists/adversarial-framework.md`. Use the Phase 2 documentation — especially the Key Invariants and User Flows sections — as your target list.

**Key activities:**
- For every invariant in Phase 2: ask "Can I break this?"
- For every user flow: "How would I extract more value than I put in?"
- For every access control entry: "Can I reach this path without the expected privilege?"
- When you find a vulnerability pattern in one function, **propagate the search** across all in-scope code.
- For every DoS/griefing vector, **analyze impact amplification** — hard caps, missing recovery paths, cascading dependencies.

---

## Step 4: Activate Domain-Specific Playbooks

Read `resources/checklists/domain-playbooks.md` to identify which playbook files match the scenarios detected in Phase 0 Step 5. Read only the matching playbook file(s) from `resources/checklists/playbooks/`. Apply each check to every relevant function.

Multiple playbooks can apply simultaneously (e.g., a lending protocol with upgradeable proxies activates both `lending-borrowing.md` and `proxy-upgradeable.md`).

---

## Step 5: Triage As You Go

For each finding — whether from a Phase 3 detector or from manual code reading — classify it immediately while context is fresh:

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Confirmed** | Real, reachable, impact clear | Queue for Phase 5 (PoC + write-up) |
| **Potential** | May be real, needs more context | Note what's needed, attempt PoC in Phase 5 |
| **Prior-Audit-Overlap** | Overlaps with a finding in `audit-output/prior-audit-catalog.md` | Do NOT dismiss — flag for severity re-evaluation (see below) |
| **False Positive** | Incorrect due to context detector can't see | Justify with specific code reference, move on |

### Handling Prior-Audit Overlaps

When a finding shares a root cause with a prior audit finding:

1. Tag it as `prior-audit-overlap` in `triage.md` and note which prior finding it maps to (by catalog #).
2. **Evaluate whether severity was underestimated** — does the prior report miss a more damaging attack path? Has the codebase changed? Can you demonstrate higher-severity impact?
3. **Verdict:** same severity + same root cause + no new attack path → out of scope. Higher severity or substantively distinct attack path → promote to Confirmed/Potential and queue for Phase 5. The write-up must reference the prior finding and explain what is new.

### Slither Finding Triage

For each Slither finding specifically:
1. Does it match something you observed during code reading?
2. Does Phase 1 structural data support it?
3. Is the code path actually reachable? (Check `call-graph`)

---

## Step 6: Verify with Foundry

Use Foundry as a **comprehension tool** during code reading:
- **Trace existing tests** (`forge test -vvvv`) to understand multi-contract flows
- **Run existing test suite** (`forge test -vvv`) — what did devs test? What did they NOT test?
- **Check coverage** (`forge coverage`) — focus manual review on uncovered critical paths

---

## Step 7: Anti-Hallucination Enforcement

Every claim about a function's behavior must be verified against `function-summary`. If your reading disagrees with the printer:
1. The printer output is correct (derived from the AST).
2. Re-read. Common causes: wrong contract (override), local vs state variable confusion, inheritance chain.
3. Only override the printer if you can point to the exact line AND explain why (e.g., inline assembly, delegatecall).

---

## Step 8: Update Phase 2 Document

If code reading reveals information that the Phase 2 codebase document is missing or got wrong — new user flows, incorrect access control claims, additional invariants, undocumented trust assumptions — update `audit-output/phase-2-docs/codebase-overview.md` now. Apply the same fact-checking standard.

---

## Step 9: Re-validate Deprioritized Hypotheses (Mandatory)

**This step is mandatory before finalizing `triage.md`.** Phase 2 deprioritized hypotheses using only code-reading-level understanding. Phase 4 has strictly more powerful tools: the code reading checklist, adversarial framework, domain playbooks, pattern propagation, and impact amplification analysis.

For each **Deprioritized** entry in `audit-output/phase-2-docs/hypothesis-list.md`:

1. **Re-read the target function** — do not rely on Phase 2 memory.
2. **Apply the full code reading checklist** — especially items the Phase 2 analysis did not consider. Pay particular attention to DoS vectors, donation attacks, and boundary values.
3. **Apply pattern propagation** — did Phase 4 discover a vulnerability pattern elsewhere that applies here?
4. **Apply impact amplification** — even if the direct effect is small, does a hard cap, missing recovery path, or cascading dependency escalate the impact?
5. **Verdict:** promote to Confirmed/Potential, or confirm as False Positive with Phase-4-level justification that references the specific checklist items and adversarial analysis that ruled it out.

Record the re-validation result and reasoning in `triage.md` for each deprioritized hypothesis.
