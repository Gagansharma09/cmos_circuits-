Custom CMOS VLSI Portfolio — SKY130

A transistor-level custom VLSI portfolio covering CMOS logic, switching circuits,
SRAM, analog building blocks, bias circuits, and a flagship 8-bit CMOS ALU.

The repository is intended to demonstrate schematic design → SPICE simulation →
characterization → PVT/variation analysis → engineering conclusions, rather than
only functional waveforms.

Technology target: SKY130 (configurable)
Primary simulator: SPICE-compatible simulator / EDAflow used for the project
Design style: transistor-level CMOS / custom VLSI
Repository status: project scaffold; simulation data and plots are added as each block is completed.

1. Portfolio Roadmap

#

Project

Main analyses

Primary parameters

01

CMOS Inverter

DC, transient, power, PVT, Monte Carlo

VTC, gain, noise margin, delay, power

02

CMOS Standard Cells

DC, transient, power, PVT

VOH/VOL, delay, rise/fall, power

03

Transmission-Gate MUX

DC, transient, power, PVT, Monte Carlo

signal degradation, delay, power

04

6T SRAM

SNM, read/write transient, power, PVT, Monte Carlo

hold/read SNM, write/read time, failure rate

05

Differential Pair / OTA

DC, AC, transient, noise, PVT, Monte Carlo

gain, UGB, phase margin, slew rate, noise

06

Current Mirror / Bias

DC, transient, PVT, Monte Carlo

current accuracy, Rout, compliance, variation

★

8-bit CMOS ALU

functional, timing, power, PVT, Monte Carlo

delay, energy/op, critical path, yield

2. Common Simulation Definitions

2.1 DC Analysis

DC analysis determines the steady-state operating point as a function of a swept
input or bias variable.

Typical sweep:

[
V_{in}: 0 \rightarrow V_{DD}
]

For an inverter:

[
V_{out}=f(V_{in})
]

Important parameters:

(V_{OH}): minimum output-high voltage

(V_{OL}): maximum output-low voltage

(V_{IH}): minimum input recognized as HIGH

(V_{IL}): maximum input recognized as LOW

(V_M): switching threshold

Voltage gain

[
A_v=\frac{dV_{out}}{dV_{in}}
]

For an inverter, the maximum magnitude of the slope is often reported as:

[
|A_{v,max}|=\max\left|\frac{dV_{out}}{dV_{in}}\right|
]

Noise margins

[
NM_H=V_{OH}-V_{IH}
]

[
NM_L=V_{IL}-V_{OL}
]

For a symmetric CMOS inverter, (V_M) is commonly obtained from:

[
V_{in}=V_{out}
]

The exact extraction method for (V_{IL}) and (V_{IH}) should be documented
in the project because practical SPICE characterization depends on the chosen
criterion.

3. Transient Analysis

Transient analysis evaluates circuit behavior versus time.

Typical digital testbench:

[
V_{in}(t)=\text{PULSE}(V_L,V_H,T_D,T_R,T_F,T_{ON},T)
]

Measure:

propagation delay

rise time

fall time

output settling

functional correctness

dynamic power

3.1 Propagation Delay

For a 50%-threshold measurement:

[
t_{pHL}=t(V_{out}\rightarrow L)-t(V_{in}\rightarrow H)
]

[
t_{pLH}=t(V_{out}\rightarrow H)-t(V_{in}\rightarrow L)
]

Average propagation delay:

[
t_p=\frac{t_{pHL}+t_{pLH}}{2}
]

3.2 Rise and Fall Time

Using 10% and 90% output levels:

[
t_r=t_{90%}-t_{10%}
]

[
t_f=t_{10%}-t_{90%}
]

The direction of the transition must be interpreted correctly for each waveform.

4. Power Analysis

Total supply power is:

[
P(t)=V_{DD}(t),I_{DD}(t)
]

Average power over a measurement window:

[
P_{avg}=
\frac{1}{T_2-T_1}
\int_{T_1}^{T_2}V_{DD}(t)I_{DD}(t),dt
]

For approximately constant supply:

[
P_{avg}\approx V_{DD}I_{DD,avg}
]

Dynamic Power

The classical CMOS approximation is:

[
P_{dyn}\approx \alpha C_L V_{DD}^{2}f
]

where:

(\alpha) = switching activity factor

(C_L) = effective switched capacitance

(V_{DD}) = supply voltage

(f) = switching frequency

Short-circuit power

During input transitions, PMOS and NMOS can conduct simultaneously:

[
P_{sc}=V_{DD}I_{sc,avg}
]

Leakage power

When the circuit is static:

[
P_{leak}=V_{DD}I_{leak}
]

For portfolio results, report:

average power

peak power where meaningful

leakage/static power

dynamic power

energy per transition/operation where applicable

5. PVT Analysis

PVT = Process, Voltage, Temperature.

Minimum recommended process corners:

TT — Typical NMOS / Typical PMOS

SS — Slow NMOS / Slow PMOS

FF — Fast NMOS / Fast PMOS

SF — Slow NMOS / Fast PMOS

FS — Fast NMOS / Slow PMOS

Recommended temperatures:

(-40^\circ C)

(27^\circ C)

(125^\circ C)

Voltage should include:

nominal (V_{DD})

low-(V_{DD})

high-(V_{DD})

Do not blindly claim every possible PVT combination. Record exactly which corners
were simulated.

PVT extraction

For each corner record:

[
t_p,\quad t_r,\quad t_f,\quad P_{avg},\quad V_{OH},\quad V_{OL}
]

For analog blocks additionally record:

[
A_v,\quad BW,\quad UGB,\quad PM,\quad SR
]

The final report should identify the best case, worst case, and nominal case.

6. Monte Carlo / Variation Analysis

Monte Carlo analysis evaluates statistical variation caused by modeled process
and/or mismatch parameters.

For a measured quantity (X), report:

Mean

[
\mu_X=\frac{1}{N}\sum_{i=1}^{N}X_i
]

Standard deviation

[
\sigma_X=
\sqrt{
\frac{1}{N-1}
\sum_{i=1}^{N}(X_i-\mu_X)^2
}
]

Coefficient of variation

[
CV=\frac{\sigma_X}{\mu_X}
]

For a normally distributed metric, an approximate (3\sigma) interval is:

[
[\mu_X-3\sigma_X,\ \mu_X+3\sigma_X]
]

For SRAM and functional circuits, do not rely only on mean/std. Report:

number of failures

failure probability

yield

worst observed sample

If (N_{pass}) samples pass:

[
Yield=\frac{N_{pass}}{N_{total}}\times100%
]

7. Project 01 — CMOS Inverter

Objective

Design and characterize a CMOS inverter at transistor level.

Required simulations

A. DC

Plot:

[
V_{out};vs.;V_{in}
]

Extract:

(V_{OH})

(V_{OL})

(V_{IL})

(V_{IH})

(V_M)

(NM_H)

(NM_L)

maximum voltage gain

B. Transient

Test repeated input transitions.

Extract:

(t_{pHL})

(t_{pLH})

average (t_p)

(t_r)

(t_f)

C. Power

Extract:

(P_{avg})

(P_{dyn})

(P_{leak})

energy/transition

D. PVT

Repeat the key DC/transient/power metrics across selected corners.

E. Monte Carlo

Report statistical variation of:

switching threshold

delay

power

8. Project 02 — CMOS Standard Cells

Cells:

NAND2

NOR2

XOR2

XNOR2

For each cell perform:

DC characterization

functional transient simulation

delay characterization

rise/fall characterization

power characterization

PVT characterization

For multi-input gates, test all input combinations relevant to the cell.

Cell comparison

Report:

Metric

NAND2

NOR2

XOR2

XNOR2

Transistor count









(t_p)









(t_r)









(t_f)









(P_{avg})









Area*









*Use a clearly defined area metric; do not mix schematic area and physical layout
area without labeling them separately.

9. Project 03 — Transmission-Gate MUX

Implement a CMOS transmission-gate 2:1 multiplexer.

Boolean function:

[
Y=\overline{S}A+SB
]

Analyses

DC

Check transfer of:

logic LOW

logic HIGH

Compare against an NMOS-only switch if included.

Transient

Test:

(S=0)

(S=1)

changing (A)

changing (B)

changing (S)

Extract:

propagation delay

rise/fall time

output voltage degradation

Power

Measure average and leakage power.

PVT

Check:

output logic levels

delay

power

Monte Carlo

Measure variation of delay and output level where practical.
