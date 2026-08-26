# Custom VLSI — Digital IC design 

A progressive, from-scratch VLSI design portfolio built around one reused digital
datapath rather than a set of unrelated circuits. Each block is verified on its
own, then reused as a subcircuit inside the flagship 8-bit ALU.

```
custom-vlsi-analog-digital-portfolio/
├── README.md
├── LICENSE
├── .gitignore
├── 01_cmos_inverter/
├── 02_cmos_standard_cells/
├── 03_transmission_gate_mux/
├── FLAGSHIP_8BIT_ALU/
├── docs/
└── scripts/
```

## Progression logic

```
CMOS inverter → standard cells → TG MUX → 8-bit ALU (flagship)
```

The inverter establishes sizing methodology and characterization workflow.
Standard cells (NAND2, NOR2, XOR2, AOI/OAI, buffer) reuse that methodology and
add functional/timing characterization at the cell-library level. The
transmission-gate MUX introduces pass-transistor logic and its distinct timing
behavior. The flagship ALU is built almost entirely out of instances of the
first three blocks (plus a ripple/carry-select adder built from the standard
cells), so its characterization is a composition exercise, not a fresh design.

---

## 01 — CMOS Inverter

**Purpose:** establish the sizing, extraction, and characterization workflow
reused by every later block.

**Design equations**

- Switching threshold (matched drive strength):
  `V_M ≈ (V_DD + V_Tn - |V_Tp| * sqrt(k_p/k_n)) / (1 + sqrt(k_p/k_n))`
- Drive strength ratio for a target V_M = V_DD/2 (unit-strength design point):
  `(W/L)_p / (W/L)_n = (μ_n / μ_p) * (unless otherwise skewed for rise/fall matching)`
- Propagation delay (RC/alpha-power approximation):
  `t_p = 0.69 * R_eq * C_L` (used only as a sanity check against SPICE)

**Required analyses**

- DC transfer characteristic (VTC) → V_IL, V_IH, V_OL, V_OH, NM_L, NM_H
- Transient propagation delay (t_pLH, t_pHL) vs. C_L sweep
- Transient rise/fall time (10–90%) vs. C_L sweep
- Static and dynamic power vs. frequency sweep
- PVT corner sweep (see Methodology)
- Monte Carlo on V_M and delay (see Methodology)

**Deliverables in this folder:** schematic, testbench, extracted netlist,
`results/` with VTC plot, delay-vs-load plot, power-vs-frequency plot, and a
short `RESULTS.md` summarizing extracted numbers against hand-calculated
targets.

---

## 02 — CMOS Standard Cells

**Purpose:** extend inverter methodology to a small library: NAND2, NOR2,
XOR2, AOI21/OAI21, and a buffer (2x inverter chain, used for driving the MUX
select lines and ALU control fan-out).

**Design equations**

- Series-stack sizing (NAND2 pull-down, NOR2 pull-up): widen the series
  devices by the stack depth to match single-transistor drive strength:
  `W_series = N * W_unit` for N devices in the worst-case series path.
- Logical effort per gate (used to size the standard-cell library
  consistently and to predict ALU critical-path delay before layout):
  `d = g*h + p`, where g = logical effort, h = electrical effort (C_out/C_in),
  p = parasitic delay.

**Required analyses**

- Functional truth-table verification (all input combinations)
- Worst-case t_pLH / t_pHL per gate (identify the worst input pattern per
  cell, not just one arbitrary vector)
- Input capacitance extraction per pin (for logical-effort based delay
  prediction later in the ALU)
- Static leakage power per cell across all input states
- PVT corner sweep
- Monte Carlo on worst-case delay

**Deliverables in this folder:** one subfolder per cell with schematic,
testbench, extracted netlist, truth-table verification log, and characterized
delay/power tables. A single `cell_library_summary.csv` aggregating all cells
for reuse in the ALU delay budget.

---

## 03 — Transmission Gate MUX

**Purpose:** introduce pass-transistor logic (2:1 and 4:1 TG MUX), used for
the ALU's operation-select datapath.

**Design equations**

- TG on-resistance vs. input voltage (why TGs are used instead of a single
  pass transistor — full-swing output, less V_T drop-dependent resistance):
  `R_on(TG) = R_on(NMOS) || R_on(PMOS)` evaluated across the input swing.
- 4:1 MUX built from 2:1 stages — the delay-through-stages tradeoff vs. a
  flat 4:1 select is documented explicitly in this block's `RESULTS.md`
  rather than assumed.

**Required analyses**

- Functional verification (all select/data combinations, 2:1 and 4:1)
- Propagation delay through worst-case TG path vs. C_L
- Select-line to output delay (control path, distinct from data path delay)
- Charge-sharing / glitch check on the shared output node during select
  transitions
- PVT corner sweep
- Monte Carlo on worst-case delay

**Deliverables in this folder:** 2:1 and 4:1 MUX schematics/testbenches,
extracted netlists, glitch-check waveform capture, and characterized delay
tables consistent in format with `02_cmos_standard_cells/cell_library_summary.csv`.

---

## FLAGSHIP — 8-bit ALU

**Purpose:** compose 01–03 into a functional 8-bit ALU (add/sub, AND, OR,
XOR, shift-left-1, pass-through), selected via the TG MUX block, built from
the characterized standard-cell library.

**What is reused vs. new**

- Full adder / carry chain: built from the standard-cell library (02)
- Operation select and output mux: built from the TG MUX block (03)
- All buffering/fan-out: sized using the inverter methodology (01)
- New in this block: the 1-bit ALU slice, the 8-bit ripple/carry-select
  composition (document which one you implement and why), and the top-level
  control decode

**Required analyses (this is a digital block — no AC/small-signal analysis
belongs here; that distinction matters for the completion criteria below)**

- Functional verification: exhaustive for the 1-bit slice, directed +
  constrained-random for the 8-bit ALU (cover all opcodes, carry-in/out
  edge cases, all-zeros/all-ones operands, overflow cases for signed add/sub)
- Critical-path timing: identify and report the worst-case opcode + operand
  pattern, not just a nominal vector; compare measured critical path against
  the logical-effort prediction assembled from `02`'s cell library summary
- Power: dynamic power vs. clock/toggle frequency, and per-opcode power
  breakdown (some opcodes toggle more of the datapath than others)
- PVT corner sweep across the full opcode set
- Monte Carlo on critical-path delay (process variation on the composed
  datapath, not per-cell in isolation)

**Deliverables in this folder:** top-level schematic, 1-bit slice
schematic, testbench (directed + constrained-random), extracted netlist,
functional coverage report, critical-path timing report (measured vs.
logical-effort-predicted), power report, PVT/Monte Carlo summary, and a
`RESULTS.md` tying every number back to the sub-block characterization it
depends on.

---

## Methodology (applies to every block above)

### Extraction parameters

- Post-layout parasitic extraction (RC, and C-only where layout is not yet
  final) documented per block in that block's `RESULTS.md`
- Corner models used for extraction and simulation listed explicitly (do not
  rely on a single "typical" model file without recording its name/version)

### PVT methodology

- **Process:** SS, TT, FF (and SF/FS for cells with asymmetric NMOS/PMOS
  sensitivity, e.g. the TG MUX)
- **Voltage:** nominal V_DD ± 10%
- **Temperature:** -40°C, 27°C, 125°C
- Report worst-case corner per metric explicitly (worst-case delay corner is
  not always the same corner as worst-case power)

### Monte Carlo statistics

- Minimum 200 runs per block (500+ for the flagship ALU critical path, given
  the larger number of composed devices)
- Report mean, σ, and 3σ for every Monte Carlo'd metric
- Both local (mismatch) and global (process) variation where the tool
  supports separating them; state which was used if only one is available

### Plot conventions

- All delay plots: x-axis load capacitance or fan-out, y-axis delay in ps,
  linear-linear unless a log-log clearly communicates a trend better
  (state which and why in the plot caption)
- All power plots: x-axis frequency, y-axis power in µW, log-x if the sweep
  spans more than one decade
- Consistent color/marker convention across blocks (defined once in
  `docs/plot_style.md`, not redefined per block)

### Completion criteria (per block)

A block is complete when its folder contains: schematic, testbench,
extracted netlist, all required analyses listed above for that block, a
`RESULTS.md` with extracted numbers compared against hand-calculated or
predicted targets, and — for any block feeding the flagship ALU — a
characterization summary in the shared CSV/table format so the ALU's
`RESULTS.md` can cite it directly instead of re-deriving it.

---

## docs/

Shared conventions referenced by every block: `plot_style.md`,
`corner_definitions.md`, `extraction_settings.md`.

## scripts/

Shared automation: corner-sweep runners, Monte Carlo post-processing,
CSV aggregation for the cell-library summary and the ALU delay-budget
comparison.
