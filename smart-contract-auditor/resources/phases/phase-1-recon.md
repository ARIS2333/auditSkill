# Phase 1: Structural Reconnaissance

**Purpose:** Build a complete, tool-verified structural model AND generate a prioritized attack hypothesis list. Do NOT read any `.sol` files during this phase.

**Gate:** You can answer every question in the table below without reading code, AND you have a ranked list of suspicious functions/contracts with specific reasons from printer output.

See `tools/slither.md` §5 for full printer syntax. Batch all printers in one command: `slither . --print p1,p2,p3,... --foundry-out-directory out`

---

## Step 1: Run All Printers

Run both structural and attack-oriented printers together:

**Structural printers:**

| Question | Printer(s) |
|----------|------------|
| What contracts exist and how big is this? | `human-summary`, `loc` |
| How do contracts relate? | `inheritance-graph`, `inheritance`, `c3-linearization` |
| What are all entry points? | `entry-points`, `function-summary` |
| What is the core state? | `variable-order`, `function-summary` |
| What constructors run at deploy? | `constructor-calls` |

**Attack-oriented printers:**

| Question | Printer(s) |
|----------|------------|
| Who can change critical state? | `vars-and-auth`, `modifiers` |
| Are pause guards complete? | `not-pausable` |
| What validation exists? | `require` |
| Any selector collisions? | `function-id` |
| Where does user input flow? | `data-dependency`, `call-graph` |

Save raw printer output files to `audit-output/phase-1-recon/printers/` (e.g., redirect DOT files, capture text output).

Record in **`audit-output/phase-1-recon/structural-summary.md`**: total contract count, SLOC, external/public functions, ERCs detected, full inheritance tree, storage layout, constructor order.

The **`function-summary`** printer is the single most important anti-hallucination artifact. It prevents you from later claiming a function reads/writes variables it does not.

---

## Step 2: Synthesize Attack Hypothesis List

Analyze all printer output together to produce a ranked target list. Write to **`audit-output/phase-1-recon/hypothesis-list.md`**.

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
