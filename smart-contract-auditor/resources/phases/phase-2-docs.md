# Phase 2: Codebase Documentation

**Purpose:** Dive into the source code using Phase 1 (structural model) as your guide, and produce a comprehensive codebase documentation with Mermaid diagrams. This document becomes the **shared reference for all subsequent phases** — it builds the mental model that drives automated scanning (Phase 3), security code reading (Phase 4), PoC development (Phase 5), and final reporting (Phase 6).

**Gate:** Document covers all 13 sections from the template. The Fact-Checking Checklist in the template passes completely. No unverified claims.

See `templates/codebase-report.md` for the full template, population guide, Mermaid examples, and fact-checking checklist.

---

## Inputs

- **Phase 1 outputs:** `audit-output/phase-1-recon/structural-summary.md`, `audit-output/phase-1-recon/hypothesis-list.md`, raw printer outputs in `audit-output/phase-1-recon/printers/`
- **Source code:** `.sol` files in scope

## Output

Write to **`audit-output/phase-2-docs/codebase-overview.md`**.

## Process

1. Open the template from `templates/codebase-report.md`
2. For each of the 13 sections, populate using the combination of printer data (Phase 1) and code reading indicated in the template's "How to Populate" table
3. Create Mermaid diagrams for architecture, user flows, access control, value flow, and state machine
4. Run through the Fact-Checking Checklist at the bottom of the template — every checkbox must pass

## Key Principles

- This is NOT a reformatting of printer output — you must go through actual source code to fill in context that printers cannot provide (protocol purpose, user flows, fee logic, state transitions, trust assumptions)
- Use printers as your guide and fact-checking source, not your only input
- Every Mermaid diagram must have labeled edges
- If you cannot fill a section confidently, run more printers or read specific contracts before proceeding — do not guess

## Living Document

This document is a **living reference** — update it when Phase 4 code reading or Phase 5 PoC development reveals new information. Every update must pass the same fact-checking standard.
