# Codebase Report — Population Guide

Read this file at the start of Phase 2 to understand which inputs feed each template section. Then switch to `codebase-report.md` to fill in the template.

---

## Using the Structural Summary

Your primary structural reference is **`audit-output/phase-1-recon/structural-summary.md`** — it already contains compact tables for the inheritance tree (section 2), function-modifier-state-write mappings (section 3), unguarded functions (section 4), and storage layout (section 5).

**Only fall back to raw printer files** (in `audit-output/phase-1-recon/printers/`) when the structural summary does not contain the detail you need. The structural summary's Printer Output Index (section 6) tells you which raw file to consult. Common reasons to access raw printers:
- DOT graph files (`inheritance-graph*.dot`, `call-graph*.dot`) for visual traversal of call paths or inheritance
- `require.txt` for per-function require/assert conditions
- `not-pausable.txt` for the full list of functions missing `whenNotPaused`
- Per-contract printer splits for large codebases (>15 contracts)

---

## Section-by-Section Input Map

| Report Section | Structural Summary Sections | Raw Printer Fallback (if needed) | Code Reading? |
|---------------|---------------------------|--------------------------------|---------------|
| 1. System Overview | — | — | Yes — README, code comments, contract names |
| 2. Project Structure | — | — | Light — `ls`/`find` + annotate |
| 3. Architecture | §2 Inheritance Tree | `inheritance-graph*.dot`, `call-graph*.dot` | Light — verify relationships |
| 4. Contract Inventory | §1 Codebase Snapshot, §2 | `human-summary.txt`, `inheritance.txt` | Light — add purpose descriptions |
| 5. Contract Deep Dives | §3 Entry Points & Access Control, §5 Storage Layout | `function-summary.txt`, `variable-order.txt`, `entry-points.txt`, `modifiers.txt` | Yes — understand function logic |
| 6. User Flows | §3 | `call-graph*.dot`, `function-summary.txt` | Yes — trace complete flows |
| 7. Access Control Matrix | §3, §4 Unguarded Functions | `vars-and-auth.txt`, `modifiers.txt` | Yes — verify guard logic |
| 8. Value Flow | §3 (filter: payable, transfer, mint, burn) | `function-summary.txt` | Yes — trace token/ETH movement |
| 9. State Machine | — | `not-pausable.txt` | Yes — identify state transitions |
| 10. External Dependencies | — | `call-graph*.dot` | Yes — identify failure modes |
| 11. Upgrade & Migration | §5 Storage Layout | `variable-order.txt` | Yes — verify storage layout |
| 12. Key Invariants | Synthesized from code reading + structural summary | — | Yes — identify protocol rules |
| 13. Known Risks & Trust Assumptions | Code reading | — | Yes — assess trust boundaries |

---

## Section-by-Section Audience Reference

| Section | Answers | Primary Audience |
|---------|---------|-----------------|
| System Overview | "What does this do?" | Everyone |
| Project Structure | "Where does the code live?" | New devs, auditors onboarding |
| Architecture | "How do contracts connect?" | New devs, auditors |
| Contract Inventory | "What am I looking at?" | New devs |
| Contract Deep Dives | "What does this contract do exactly?" | Devs working on specific contracts |
| User Flows | "What happens when a user does X?" | Frontend devs, integrators, auditors |
| Access Control | "Who can do what?" | Security reviewers, ops team |
| Value Flow | "How does money move?" | Auditors, economists |
| State Machine | "What states can the system be in?" | Ops team, incident responders |
| External Dependencies | "What can break us from outside?" | Risk team, auditors |
| Upgrade & Migration | "How does upgrading work?" | Auditors, ops team |
| Key Invariants | "What must always be true?" | Auditors, fuzz test writers |
| Known Risks | "What are we trusting?" | Everyone |

---

## Key Principles

- This is NOT a reformatting of Slither printer output — go through the actual source code to fill in context that printers cannot provide (protocol purpose, user flows, fee logic, state transitions, trust assumptions).
- Replace all `[bracketed placeholders]` with actual values. Delete sections that do not apply.
- Every Mermaid diagram must have labeled edges.
- Every section should be independently useful — a developer can jump to any section without reading prior ones.
- Use tables for structured data, diagrams for relationships, prose only for context.
- Document all unhappy paths (edge cases, failure modes, what happens when things break).
- Record the commit hash — stale docs are worse than no docs.
