# Phase 1: Structural Reconnaissance

**Purpose:** Build a complete, tool-verified structural model of the codebase using Slither printers. Do NOT read any `.sol` files during this phase.

**Gate:** You can answer every structural question in the table below without reading code, AND the structural summary includes a printer output guide.

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

## Step 2: Write Structural Summary

Write **`audit-output/phase-1-recon/structural-summary.md`** using the template in `templates/structural-summary.md`. This file is a quick codebase snapshot plus an index to all printer output files — do NOT copy detailed printer output into it.
