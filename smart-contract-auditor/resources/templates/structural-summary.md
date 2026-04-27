# Structural Summary Template

Use this template for `audit-output/phase-1-recon/structural-summary.md`. Fill in values from printer output. Do NOT copy detailed printer output into this file — it is an index, not a dump.

---

```markdown
# Structural Summary

## Codebase Snapshot

| Metric | Value |
|--------|-------|
| Total contracts (in scope) | [N] |
| Source lines of code (SLOC) | [N] |
| External/public functions | [N] |
| ERCs detected | [e.g., ERC20, ERC721, None] |

**Source:** `printers/human-summary.txt`

## Printer Output Index

All files are in `audit-output/phase-1-recon/printers/`.

| File | What to Find Here |
|------|-------------------|
| `human-summary.txt` | Contract count, SLOC, ERCs, complexity overview |
| `function-summary.txt` | **Anti-hallucination ground truth.** Per-function: visibility, modifiers, state vars read/written |
| `entry-points.txt` | All state-changing external/public functions and their accessed variables |
| `vars-and-auth.txt` | Which state variables each function modifies + authorization checks |
| `inheritance.txt` | Text-based inheritance relationships between contracts |
| `inheritance-graph*.dot` | DOT visualization of inheritance hierarchy |
| `variable-order.txt` | Storage slot layout — variable ordering per contract |
| `modifiers.txt` | Which modifiers are applied to each function |
| `require.txt` | All `require` and `assert` conditions per function |
| `not-pausable.txt` | Functions missing `whenNotPaused` guard |
| `call-graph*.dot` | DOT visualization of inter-function call relationships |
| `data-dependency.txt` | How user input flows through to state variables |
| `function-id.txt` | Keccak256 function selectors (check for collisions in proxy patterns) |

## Notes

[Record anything unusual observed during printer execution: errors, warnings, contracts that failed to analyze, printers that produced empty output, etc.]
```
