# Phase 1: Structural Reconnaissance

**Purpose:** Build a complete, tool-verified structural model of the codebase using Slither printers. Do NOT read any `.sol` files during this phase.

**Gate:** You can answer every structural question (contract count, inheritance hierarchy, function visibility, storage layout, access control modifiers, entry points) without reading code, the structural summary includes a printer output guide, all printer outputs are validated as non-empty and error-free, AND a preliminary hypothesis list is produced.

---

## Step 1: Run All Printers (Each to Its Own File)

Run 13 printers, saving each output to a separate file in `audit-output/phase-1-recon/printers/`. Always write the full `slither` command directly — never store commands in shell variables.

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
  variable-order modifiers require not-pausable data-dependency function-id; do
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

After all 13 printers have run, verify that each output file is usable. Non-empty output and absence of error prefixes are required before proceeding.

```bash
for f in audit-output/phase-1-recon/printers/*.txt; do
  filename=$(basename "$f")

  # Check non-empty
  if [ ! -s "$f" ]; then
    echo "FAIL: $filename is empty"
    continue
  fi

  # Check for error indicators at start of file
  head -5 "$f" | grep -qi "error\|traceback\|exception\|not found" && \
    echo "WARNING: $filename may contain an error — review manually"
done

# Verify DOT files exist
ls audit-output/phase-1-recon/printers/*.dot >/dev/null 2>&1 || echo "FAIL: No DOT files found"
```

**Pay special attention to:**
- **`function-summary.txt`** — this is the anti-hallucination ground truth. If it is empty or contains errors, **stop and fix before proceeding**. Common cause: Slither failed to compile one or more contracts silently.
- **`vars-and-auth.txt`** — if empty, access control analysis in Phase 2 will be unreliable.
- **DOT files** (`*.dot`) — verify at least one inheritance graph and one call graph file exist.

If any critical printer output is missing or broken, return to Phase 0.3 to diagnose the Slither issue. Document any printers that could not produce output in the structural summary notes.

---

## Step 3: Run Test Coverage (If Tests Exist)

If the project has test files (`test/` directory is non-empty), run `forge coverage` to identify which code paths the existing test suite exercises:

```bash
forge coverage > audit-output/phase-1-recon/coverage-report.txt 2>&1
```

If `forge coverage` fails (e.g., tests require RPC forking or have unmet dependencies), note the failure and move on — this is informational, not blocking.

**Use the coverage output to inform the hypothesis list:** functions with zero or low coverage in critical contracts are higher-priority targets. If coverage shows that certain user flows are completely untested, note those in the preliminary hypothesis list.

---

## Step 4: Write Structural Summary

Write **`audit-output/phase-1-recon/structural-summary.md`** using the template in `templates/structural-summary.md`.

This is NOT just an index — it extracts the most critical structural data into compact tables so that later phases can look up facts here without re-reading large raw printer files. The template has 7 sections:

1. **Codebase Snapshot** — metrics from `human-summary.txt`
2. **Inheritance Tree** — compact text hierarchy from `inheritance.txt`
3. **Entry Points & Access Control** — the most important table: every external/public state-modifying function with its modifiers, state writes, and auth checks. Extracted from `function-summary.txt` + `modifiers.txt` + `vars-and-auth.txt`.
4. **Unguarded State-Modifying Functions** — cross-referenced from `function-summary.txt` + `modifiers.txt` + `require.txt`. Pre-identified for the hypothesis list.
5. **Storage Layout Overview** — per-contract slot layout from `variable-order.txt`
6. **Printer Output Index** — pointers to raw files for detail lookups
7. **Notes** — validation failures, warnings, coverage highlights

For each section, read the listed printer file(s), extract the data into the structured format, then move on. Do NOT copy raw printer output verbatim — summarize into the tables. The goal: anyone reading this file should understand the codebase structure without opening any raw printer file.

---

## Step 5: Produce Preliminary Hypothesis List

Using ONLY the printer output (no `.sol` file reading), produce a preliminary hypothesis list. Write to **`audit-output/phase-1-recon/preliminary-hypotheses.md`**.

Scan the printer output for these structural signals:

| Signal | Source Printer | Priority |
|--------|---------------|----------|
| Functions with NO modifiers AND state-modifying | `function-summary.txt`, `modifiers.txt` | Critical |
| Functions with NO `require`/`assert` checks AND state-modifying | `require.txt`, `function-summary.txt` | Critical |
| Functions handling ETH transfers (payable, `.call{value:}`, `.transfer`) | `function-summary.txt` | High |
| Functions handling token transfers (`transferFrom`, `safeTransferFrom`, `mint`, `burn`) | `function-summary.txt` | High |
| Complex cross-contract calls (multiple external calls in one function) | `call-graph*.dot`, `entry-points.txt` | High |
| State-modifying functions missing `whenNotPaused` (if Pausable is used) | `not-pausable.txt` | Medium |
| Functions with high data dependency from user input | `data-dependency.txt` | Medium |
| Zero or low test coverage (if coverage was run in Step 3) | `coverage-report.txt` | Medium |

Format:

```markdown
# Preliminary Hypothesis List (Phase 1 — Structural Only)

> This list is based solely on printer output. Phase 2 will refine it with code-level understanding.

| # | Target (Contract.function) | Signal | Priority | Coverage |
|---|---------------------------|--------|----------|----------|
| 1 | [Contract.function()] | [No modifiers, state-modifying] | Critical | [Untested / N/A] |
```

This list is intentionally coarse — it identifies *where* to look, not *what* the bug is. Phase 2 code reading will validate, refine, and re-rank these hypotheses.
