# Phase 1: Structural Reconnaissance

**Purpose:** Build a complete, tool-verified structural model of the codebase using Slither printers. Do NOT read any `.sol` files during this phase.

**Gate:** You can answer every structural question (contract count, inheritance hierarchy, function visibility, storage layout, access control modifiers, entry points) without reading code. Structural summary includes a printer output guide. All printer outputs validated as non-empty and error-free.

**Inputs:** Compiled codebase, Slither base flags from Phase 0.

**Outputs:**
- `audit-output/phase-1-recon/printers/` — one file per printer (12 total)
- `audit-output/phase-1-recon/structural-summary.md`
- `audit-output/phase-1-recon/coverage-report.txt` (if tests exist)

**Checkpoint:** Present the structural summary to the user. Pause and wait for confirmation before proceeding to Phase 2.

---

## Step 1: Run All Printers

Run 12 printers, saving each output to a separate file in `audit-output/phase-1-recon/printers/`. Always write the full `slither` command directly — never store commands in shell variables.

**Text printers** — run individually, redirect stdout to file:

```bash
# First printer compiles the project
slither . --print human-summary \
  --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  > audit-output/phase-1-recon/printers/human-summary.txt 2>&1

# Remaining text printers skip recompilation
for printer in function-summary entry-points vars-and-auth inheritance \
  variable-order modifiers require not-pausable function-id; do
  slither . --print "$printer" \
    --foundry-out-directory out \
    --exclude-dependencies \
    --filter-paths 'lib|node_modules|test|script' \
    --foundry-ignore-compile \
    > "audit-output/phase-1-recon/printers/${printer}.txt" 2>&1
done
```

**DOT printers** — these auto-generate `.dot` files in the working directory:

```bash
slither . --print inheritance-graph,call-graph \
  --foundry-out-directory out \
  --exclude-dependencies \
  --filter-paths 'lib|node_modules|test|script' \
  --foundry-ignore-compile

mv *.dot audit-output/phase-1-recon/printers/ 2>/dev/null
```

**Note:** The `--filter-paths` value uses `|` for regex alternation — it MUST be wrapped in single quotes to prevent the shell from interpreting `|` as a pipe operator.

---

## Step 2: Validate Printer Outputs

Verify that each output file is usable. Non-empty output and absence of error prefixes are required before proceeding.

```bash
for f in audit-output/phase-1-recon/printers/*.txt; do
  filename=$(basename "$f")

  if [ ! -s "$f" ]; then
    echo "FAIL: $filename is empty"
    continue
  fi

  head -5 "$f" | grep -qi "error\|traceback\|exception\|not found" && \
    echo "WARNING: $filename may contain an error — review manually"
done

ls audit-output/phase-1-recon/printers/*.dot >/dev/null 2>&1 || echo "FAIL: No DOT files found"
```

**Pay special attention to:**
- **`function-summary.txt`** — this is the anti-hallucination ground truth. If it is empty or contains errors, **stop and fix before proceeding**. Common cause: Slither failed to compile one or more contracts silently.
- **`vars-and-auth.txt`** — if empty, access control analysis in Phase 2 will be unreliable.
- **DOT files** (`*.dot`) — verify at least one inheritance graph and one call graph file exist.

If any critical printer output is missing or broken, return to Phase 0 Step 4 to diagnose. Document any printers that could not produce output in the structural summary notes.

---

## Step 3: Run Test Coverage (If Tests Exist)

If the project has test files (`test/` directory is non-empty):

```bash
forge coverage > audit-output/phase-1-recon/coverage-report.txt 2>&1
```

If `forge coverage` fails (e.g., tests require RPC forking), note the failure and move on — this is informational, not blocking.

Use coverage output to inform Phase 3 attack planning: functions with zero or low coverage in critical contracts are higher-priority targets.

---

## Step 4: Write Structural Summary

Write **`audit-output/phase-1-recon/structural-summary.md`** using the template in `resources/templates/structural-summary.md`.

This is NOT just an index — extract the most critical structural data into compact tables so later phases can look up facts here without re-reading large raw printer files. The template has 7 sections:

1. **Codebase Snapshot** — metrics from `human-summary.txt`
2. **Inheritance Tree** — compact text hierarchy from `inheritance.txt`
3. **Entry Points & Access Control** — every external/public state-modifying function with modifiers, state writes, and auth checks. Extracted from `function-summary.txt` + `modifiers.txt` + `vars-and-auth.txt`.
4. **Unguarded State-Modifying Functions** — cross-referenced from `function-summary.txt` + `modifiers.txt` + `require.txt`. Pre-identified as high-priority attack targets for Phase 3.
5. **Storage Layout Overview** — per-contract slot layout from `variable-order.txt`
6. **Printer Output Index** — pointers to raw files for detail lookups
7. **Notes** — validation failures, warnings, coverage highlights

For each section: read the listed printer file(s), extract the data into the structured format, then move on. Do NOT copy raw printer output verbatim — summarize into the tables.

