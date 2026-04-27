# Phase 2: Codebase Documentation

**Purpose:** Dive into the source code using Phase 1 (structural model) as your guide, and produce a comprehensive codebase documentation with Mermaid diagrams. This document becomes the **shared reference for all subsequent phases** — it builds the mental model that drives automated scanning (Phase 3), security code reading (Phase 4), PoC development (Phase 5), and final reporting (Phase 6).

**Gate:** Document covers all 13 sections from the template. The Fact-Checking Checklist in the template passes completely. No unverified claims. A ranked hypothesis list is produced at the end.

See `templates/codebase-report.md` for the full template, population guide (which printer files to read per section), and fact-checking checklist.

---

## Inputs

- **Phase 1 outputs:** `audit-output/phase-1-recon/structural-summary.md` (includes printer output index), raw printer outputs in `audit-output/phase-1-recon/printers/`
- **Phase 1 output:** `audit-output/phase-1-recon/preliminary-hypotheses.md` (structural-only hypothesis list to refine)
- **Phase 1 output:** `audit-output/phase-1-recon/coverage-report.txt` (if tests exist)
- **Source code:** `.sol` files in scope

## Outputs

- **`audit-output/phase-2-docs/codebase-overview.md`** — comprehensive codebase documentation
- **`audit-output/phase-2-docs/hypothesis-list.md`** — ranked attack hypothesis list

## Process

1. Open the template from `templates/codebase-report.md`
2. For each of the 13 sections, follow the template's "Read These Files Before Writing" column — **read the listed printer files fresh for each section, do not rely on memory**
3. Run through the Fact-Checking Checklist at the bottom of the template — every checkbox must pass
4. **Refine the attack hypothesis list** — start from `audit-output/phase-1-recon/preliminary-hypotheses.md` and update with code-level understanding (see below)

This document is a **living reference** — update it when Phase 4 code reading or Phase 5 PoC development reveals new information.

**Context management:** Start each section by reading the Phase 1 structural summary for a quick orientation — the compact function reference and inheritance tree cover most lookups. Then read the specific raw printer file(s) listed in the template's "Read These Files Before Writing" column for full detail on that section. After writing each section to the codebase overview, subsequent sections can reference what you already wrote rather than re-reading the same raw printer data. This "structural summary → raw printer for detail → write section → move on" cycle keeps context manageable.

---

## Final Step: Refine Attack Hypothesis List

After completing the codebase documentation, you now have both the structural model (Phase 1 printers) and deep protocol understanding (code reading). Start from the Phase 1 preliminary hypothesis list (`audit-output/phase-1-recon/preliminary-hypotheses.md`) and refine it: validate or invalidate each structural signal with code-level context, re-rank based on actual understanding, and add new hypotheses discovered during code reading.

Write to **`audit-output/phase-2-docs/hypothesis-list.md`**.

**Flag immediately:**
- Functions that modify state with NO modifiers and NO require checks (critical priority)
- Functions with only `msg.sender` checks (medium priority)
- State-modifying functions missing `whenNotPaused` (if project uses Pausable)

| Priority | Criteria |
|----------|----------|
| **Critical** | State-modifying, no modifiers, no require statements |
| **High** | Handles ETH/token transfers with access control (verify guard correctness) |
| **Medium** | Complex functions with many state writes and cross-function calls |
| **Low** | View functions, well-guarded administrative functions |

This refined hypothesis list now benefits from both structural data AND your code reading. Compare it against the Phase 1 preliminary list — note which structural signals were confirmed, which were false alarms, and what new targets emerged from code reading.
