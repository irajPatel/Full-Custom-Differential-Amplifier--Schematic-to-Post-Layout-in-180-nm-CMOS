# Differential Amplifier: UMC 180 nm CMOS

A single-ended-output differential amplifier, designed, simulated, laid out and verified in **Cadence Virtuoso** on the **UMC 180 nm** process, from schematic through post-layout simulation.

Completed as part of the **TTRP Project**.

| | |
| --- | --- |
| **Team** | Peaky Balwindars |
| **Members** | Ravi Patel, Kartik Jhangir, Rupesh, Suryanshu Raghav |
| **Library** | `TTRP_RaviRupesh_kartik` |
| **Cell** | `diff_amp` |

---

## Contents

1. [Introduction](#1-introduction)
2. [Theory of operation](#2-theory-of-operation)
3. [Key equations](#3-key-equations)
4. [Design specifications](#4-design-specifications)
5. [Schematic](#5-schematic)
6. [Pre-layout simulation](#6-pre-layout-simulation)
7. [Layout, DRC and LVS](#7-layout-drc-and-lvs)
8. [Parasitic extraction and post-layout simulation](#8-parasitic-extraction-and-post-layout-simulation)
9. [Pre-layout vs post-layout comparison](#9-pre-layout-vs-post-layout-comparison)
10. [Final results](#10-final-results)
11. [Reproducing the design](#11-reproducing-the-design)
12. [Project structure](#12-project-structure)

---

## 1. Introduction

A **differential amplifier** has two inputs and produces an output that depends on their **difference**:

$$v_{id} = v_{G1} - v_{G2}$$

It amplifies signals that appear *between* the inputs (differential mode) and rejects signals that appear on *both* inputs equally (common mode), such as supply ripple, hum and interference.

| Input condition | Desired output |
| --- | --- |
| Inputs move in opposite directions (signal) | Large output |
| Inputs move in the same direction (interference) | Almost none |

This circuit is the input stage of op-amps, comparators and sense amplifiers. In this design the inputs are pins **A** and **B** (the gates of the NMOS pair) and the output `OUT` is single-ended, measured with respect to ground.

---

## 2. Theory of operation

### 2.1 Circuit blocks

| Block | Devices | Function |
| --- | --- | --- |
| Differential pair | NMOS M1, M2 | Converts $v_{id}$ into a difference of drain currents |
| Tail current source | NMOS M0 | Sinks a fixed current $I_{SS}$ shared by M1 and M2 |
| Active load | PMOS current mirror | Folds the current difference onto one node, giving a single-ended output |

Supply: **1.8 V** between `VDD` and `VSS`.

### 2.2 Common-mode and differential-mode inputs

Any input pair is decomposed as

$$v_{ic} = \frac{v_{G1}+v_{G2}}{2}, \qquad v_{id} = v_{G1}-v_{G2}$$

$$v_{G1} = v_{ic} + \frac{v_{id}}{2}, \qquad v_{G2} = v_{ic} - \frac{v_{id}}{2}$$

### 2.3 Balanced condition ($v_{id}=0$)

M1 and M2 are identical, so the tail current splits equally:

$$I_{D1} = I_{D2} = \frac{I_{SS}}{2}$$

No net current reaches the output, and $v_{OUT}$ stays at its bias point.

### 2.4 Differential input

Raising $v_{G1}$ by $v_{id}/2$ and lowering $v_{G2}$ by $v_{id}/2$ steers current from M2 to M1. The tail current $I_{SS}$ is fixed, so the changes are equal and opposite:

$$i_{d1} = +g_m\frac{v_{id}}{2}, \qquad i_{d2} = -g_m\frac{v_{id}}{2}$$

The shared source node does not move for a purely differential input (virtual ground), so each device sees the full half-difference as its $v_{gs}$.

The PMOS mirror copies $i_{d1}$ to the output branch, where it is compared with $i_{d2}$. The two contributions add:

$$i_{out} = \left(\frac{I_{SS}}{2}+g_m\frac{v_{id}}{2}\right)-\left(\frac{I_{SS}}{2}-g_m\frac{v_{id}}{2}\right)=g_m\,v_{id}$$

The mirror therefore recovers the full $g_m v_{id}$, about twice the gain of a pair with a plain resistor load on one side only.

### 2.5 Common-mode input

If both gates move together, the shared source follows them, so $v_{gs}$ of M1 and M2 barely changes. Both branches keep carrying about $I_{SS}/2$ and $v_{OUT}$ stays nearly constant. A real tail has finite output resistance, so a small residual common-mode gain remains. A higher tail output resistance (longer channel, cascode) and better M1/M2 matching both improve rejection.

### 2.6 Saturation and input range

The small-signal model holds only while every transistor is in **saturation**:

- M1, M2 need sufficient $V_{DS}$ above the tail node.
- The PMOS mirror needs sufficient $V_{SD}$ below `VDD`.
- The tail M0 must stay saturated so $I_{SS}$ is constant.

The range of $v_{ic}$ satisfying all three is the **input common-mode range**. Outside it, one side of the pair takes all of $I_{SS}$ and the output flattens (visible in the DC sweep, Section 8).

---

## 3. Key equations

### 3.1 Tail current

$$I_{D1}+I_{D2}=I_{SS}$$

### 3.2 Transconductance

Each input device is biased at $I_D = I_{SS}/2$:

$$g_m=\mu_n C_{ox}\frac{W}{L}V_{OV}=\frac{2I_D}{V_{OV}}=\sqrt{2\mu_n C_{ox}\frac{W}{L}I_D},\qquad V_{OV}=V_{GS}-V_{TH}$$

### 3.3 Output resistance

$$R_{out}=r_{o2}\parallel r_{o4}$$

where $r_{o2}$ is the output resistance of the NMOS (M2) and $r_{o4}$ that of the output PMOS. The PMOS load uses $L = 720\,\text{nm}$, twice the input-pair length, to raise $r_o$ and hence $R_{out}$.

### 3.4 Differential gain

$$A_{dm}=\frac{v_{out}}{v_{id}}=g_m\,R_{out}=g_m\,(r_{o2}\parallel r_{o4})$$

$$A_{dm,\text{dB}}=20\log_{10}|A_{dm}|$$

### 3.5 Common-mode gain and CMRR

$$A_{cm}=\frac{v_{out}}{v_{ic}}, \qquad \text{CMRR}=\left|\frac{A_{dm}}{A_{cm}}\right|, \qquad \text{CMRR}_{\text{dB}}=20\log_{10}\left|\frac{A_{dm}}{A_{cm}}\right|$$

### 3.6 Bandwidth (single dominant pole)

$$f_p\approx\frac{1}{2\pi R_{out}C_{out}}$$

$C_{out}$ includes device junction capacitance and, after layout, wire capacitance. Extraction is therefore expected to lower $f_p$.

### 3.7 Power

$$P = V_{DD}\cdot I_{supply}\approx 1.8\,\text{V}\times 40\,\mu\text{A}\approx 72\,\mu\text{W}$$

The 20 µA reference branch and the 20 µA tail branch (1:1 mirror) each draw from the supply.

### 3.8 Parameter summary

| Parameter | Meaning | Set by |
| --- | --- | --- |
| $A_{dm}$ | Output volts per volt of difference | $g_m$ and $R_{out}$ |
| $g_m$ | Voltage-to-current conversion of one input device | $I_{SS}/2$, $W/L$ |
| $R_{out}$ | Voltage developed by $i_{out}$ | PMOS/NMOS $r_o$ (channel length, current) |
| CMRR | Rejection of shared disturbances | Tail resistance, M1–M2 matching |
| Input CM range | $v_{ic}$ window keeping all devices saturated | Tail and pair headroom |
| Bandwidth | Frequency where gain begins to fall | $R_{out}C_{out}$ |

---

## 4. Design specifications

| Item | Value |
| --- | --- |
| Process | UMC 180 nm CMOS (`UMC_18_CMOS`) |
| Devices | `N_18_MM` (NMOS), `P_18_MM` (PMOS) |
| Supply | 1.8 V |
| Temperature | 27 °C |
| Tail bias | 20 µA reference |
| Input pair M1, M2 | NMOS, W = 1.44 µm, L = 360 nm |
| PMOS mirror load | PMOS, W = 1.44 µm, L = 720 nm |
| Tail M0 | NMOS, W = 3.6 µm, L = 360 nm |
| Output | Single-ended (`OUT`) |
| Tools | Cadence Spectre (ADE L), Assura DRC/LVS, Quantus QRC |

**Analyses used for every electrical check**

| Analysis | Setting | Purpose |
| --- | --- | --- |
| Transient | Stop time 5 ms | Output waveform for a small differential sine |
| AC | 100 Hz – 150 MHz, 20 points/decade | Small-signal gain and roll-off |
| DC | One input swept across the supply | Large-signal transfer curve and bias region |

---

## 5. Schematic

![Transistor schematic of diff_amp](Image/Diff_amp_design_step1.png)

*Figure 1. Schematic of `diff_amp`: NMOS pair (centre), PMOS mirror (toward `VDD`), tail device (to `VSS`). Pins A and B are the inputs; `OUT` is taken from the non-diode-connected side of the mirror.*

| Device | Role | Model | W | L |
| --- | --- | --- | --- | --- |
| M1, M2 | Input pair | `N_18_MM` | 1.44 µm | 360 nm |
| PMOS pair | Mirror load | `P_18_MM` | 1.44 µm | 720 nm |
| M0 | Tail current source | `N_18_MM` | 3.6 µm | 360 nm |

A second NMOS forms the bias branch that sets the gate voltage of M0 from the 20 µA reference.

**Checks performed:** matched input devices, matched PMOS loads, tail in series with the shared source, correct output node, supplies on `VDD`/`VSS`. Every later stage is derived from this schematic, so it acts as the specification for the layout.

### Circuit operation (one half-cycle)

1. A rises and B falls by the same small amount; the common mode stays near 0.9 V.
2. The left NMOS takes more tail current, the right NMOS takes less.
3. The diode-connected PMOS carries the larger current and sets the mirror gate voltage.
4. The second PMOS copies that current into the output branch.
5. The right NMOS sinks less than the PMOS sources, so the excess charges `OUT` and $v_{OUT}$ rises.

The opposite half-cycle reverses the sequence. A sine at the input therefore gives a sine at the output at the same frequency.

### Testbench sources

| Source | Value | Purpose |
| --- | --- | --- |
| `vdc` | 1.8 V | Core supply |
| `idc` | 20 µA | Tail bias reference |
| `vsin` | 5 mV amplitude, common mode ≈ 0.9 V | Small differential sine (keeps the circuit in the linear region) |

---

## 6. Pre-layout simulation

Runs the schematic netlist with ideal wires. State saved as `spectre_state1` (transient, AC and DC enabled, Spectre, 27 °C).

![ADE L testbench before layout](Image/Pre_layout_sim_start_part1.png)

*Figure 2. Pre-layout testbench and ADE L setup.*

![Pre-layout transient and AC magnitude](Image/Prelayout_sim_plot1.png)

*Figure 3. Pre-layout results. Left: transient inputs and output (5 ms). Right: AC magnitude (100 Hz – 150 MHz).*

![Pre-layout gain in dB](Image/Pre_layout_sim_plot2.png)

*Figure 4. Pre-layout gain, `dB20(VF("/out"))`.*

**Observations**

- **Transient:** inputs are equal-amplitude, opposite-phase sines centred near 0.9 V; `OUT` is a sine at the same frequency, as predicted in Section 2.4.
- **AC:** flat passband (the value of $A_{dm}$) followed by a smooth single-pole roll-off with no peaking, consistent with Section 3.6.
- **Purpose:** confirms the schematic amplifies correctly and provides the reference for the post-layout comparison.

---

## 7. Layout, DRC and LVS

### 7.1 Layout

![Layout of diff_amp](Image/Diff_amp_Layout.png)

*Figure 5. Layout of `diff_amp`. PMOS loads along the top, input pair in the middle, tail devices along the bottom; Metal1 straps carry `VDD`, `VSS`, the inputs and `OUT`.*

The input pair is placed as a matched couple so that $I_{D1}=I_{D2}$ when $v_{id}=0$. Net names (`VDD`, `VSS`, A, B, `OUT`) are carried over from the schematic.

### 7.2 DRC (Assura)

Design-rule check verifies width, spacing and enclosure rules of the UMC 180 nm deck.

![DRC result](Image/Diff_amp_DRC_check.png)

*Figure 6. Assura DRC: "No DRC errors found."*

### 7.3 LVS (Assura, run `exp1`)

LVS confirms the layout implements the same circuit as the schematic.

![LVS result](Image/Diff_amp_LVS_check.png)

*Figure 7. Assura LVS: schematic and layout match.*

| LVS item | Result |
| --- | --- |
| Schematic vs layout | Match |
| Malformed devices, shorts, opens | None |
| Net / device / pin / parameter mismatches | 0 / 0 / 0 / 0 |
| DRC violations reported with the run | 0 |

DRC guarantees manufacturability; LVS guarantees functional equivalence. Both must pass before parasitics are extracted.

---

## 8. Parasitic extraction and post-layout simulation

### 8.1 Extraction (Quantus QRC, run `exp1`)

RC-decoupled extraction with `VSS` as the ground net, using the UMC 180 nm LPE deck. Output view: `av_extracted`.

![Quantus extraction completed](Image/Diff_amp_RC_extraction.png)

*Figure 8. Quantus QRC completed and created `diff_amp/av_extracted`.*

| Extracted instance | Count | Description |
| --- | --- | --- |
| `N_18_MM` | 4 | Input pair, tail and bias NMOS |
| `P_18_MM` | 2 | PMOS mirror load |
| `presistor` | 51 | Parasitic wire and contact resistance |
| `pcapacitor` | 35 | Parasitic capacitance to ground |

### 8.2 Binding the testbench to the extracted view

A **config** view makes the testbench instance of `diff_amp` use `av_extracted` while the ideal sources keep their Spectre views. Without it, re-netlisting would simulate the schematic again. The config is stored in `temp_test/config`, with instance `I0` bound to `av_extracted`.

| | |
| --- | --- |
| ![Step 1](Image/Post_layout_step1.png) | ![Step 2](Image/Post_layout_step2.png) |
| *Figure 9. Open the testbench cell.* | *Figure 10. Create the config view.* |

![Config bound to the extracted view](Image/Post_layout_step3.png)

*Figure 11. Hierarchy Editor binding `diff_amp` to `av_extracted`.*

### 8.3 Post-layout results

![Post-layout transient, AC and DC](Image/Post_layout_plot1.png)

*Figure 12. Post-layout results. Top left: transient. Top right: AC magnitude. Bottom left: DC transfer curve.*

- **Transient:** opposite-phase input sines give a sinusoidal `OUT` at the same frequency; the layout preserves the differential action.
- **AC:** flat passband then a single-pole roll-off, now including extracted $C_{out}$.
- **DC sweep:** steep region where both sides of the pair are saturated (small-signal model valid), flattening where one device takes all of $I_{SS}$. This shows the input range of Section 2.6.

![Post-layout gain with marker](Image/post_layout_plot2.png)

*Figure 13. Post-layout gain, `dB20(VF("/out"))`, with the passband marker at **−40.196 dB**.*

The marker gives the post-layout passband value of `dB20(VF("/out"))`:

$$10^{-40.196/20}\approx 0.0098$$

This equals $|A_{dm}|$ only if the AC stimulus magnitude is 1 V (see the note in Section 10).

---

## 9. Pre-layout vs post-layout comparison

Same testbench, same analyses, same gain definition. Only the view under the symbol changes: `schematic` → `av_extracted`.

| Check | Pre-layout | Post-layout |
| --- | --- | --- |
| Netlist | Schematic devices only | Devices + 51 resistors + 35 capacitors |
| Transient | Opposite-phase inputs, sinusoidal `OUT` | Same behaviour |
| AC shape | Flat passband, single roll-off | Flat passband, single roll-off |
| Gain marker | Flat passband at low frequency | **−40.196 dB** |
| DC transfer | Steep region, flat outside | Steep region, flat outside |
| Sources of change | Model device capacitance only | Added wire R and C |

Since LVS matched, any shift between the two runs is due to the extracted parasitics: extra $C_{out}$ lowers $f_p$, and series resistance slightly alters $R_{out}$ and $A_{dm}$.

---

## 10. Final results

| Result | Value |
| --- | --- |
| Process / supply | UMC 180 nm / 1.8 V |
| Topology | NMOS pair, NMOS tail, PMOS mirror load, single-ended output |
| Input pair | 1.44 µm / 360 nm |
| PMOS load | 1.44 µm / 720 nm |
| Tail (M0) | 3.6 µm / 360 nm |
| Extracted devices | 4 × `N_18_MM`, 2 × `P_18_MM` |
| Bias / temperature | 20 µA reference / 27 °C |
| Estimated power | ≈ 72 µW |
| DRC | No errors |
| LVS | Match, 0 mismatches |
| Extraction | `av_extracted`, RC-decoupled, Quantus run `exp1` |
| Post-layout passband marker | **−40.196 dB** (`dB20(VF("/out"))`) |
| Frequency response | Flat passband, single-pole roll-off |

> **Note on the gain value.** `dB20(VF("/out"))` equals the voltage gain in dB only when the AC source magnitude is 1 V. If the AC magnitude in the testbench is different (for example 5 mV), the true gain is the marker value minus $20\log_{10}$ of that magnitude. Please confirm the AC magnitude setting before quoting −40.196 dB as $A_{dm}$.

### Conclusion

Two matched NMOS transistors share a tail current, so a gate-voltage difference becomes a drain-current difference. The PMOS mirror converts it to a single-ended output, $v_{OUT}=g_mR_{out}v_{id}$, while all devices remain in saturation. The design was carried through a full 180 nm flow: pre-layout simulation showed the expected differential and single-pole behaviour, the layout passed DRC and LVS, and the extracted view preserved the same behaviour with parasitics included.

---

## 11. Reproducing the design

### Requirements

- Cadence Virtuoso IC 6.1.8 (or a nearby IC 6.1 release) and Spectre
- UMC 180 nm design kit: `UMC_18_CMOS`, Assura rule decks, Quantus LPE deck
- Cadence `analogLib` and `basic` libraries

> Stale `*.cdslck` lock files may be present from the lab session. If a cell opens read-only, close other sessions and delete the lock files in that cell directory.

### Step 1: Register the libraries

Add to your `cds.lib`:

```text
DEFINE TTRP_RaviRupesh_kartik  /path/to/Ravi_diff
DEFINE UMC_18_CMOS             /path/to/UMC180/UMC_18_CMOS
```

The model section used is `mm180_reg18` (typical), under the kit's `Models/Spectre` directory.

### Step 2: Open the schematic

```bash
virtuoso &
```

In Library Manager, open `TTRP_RaviRupesh_kartik` → `diff_amp` → `schematic` (compare with Figure 1).

### Step 3: Pre-layout simulation

1. Open the testbench schematic (`temp_test`, or `test_diff_amp` if present).
2. Confirm sources: 1.8 V supply, 20 µA bias, 5 mV input sine, common mode ≈ 0.9 V.
3. Launch **ADE L** and load `spectre_state1` (**Session → Load State**).
4. Verify analyses: transient `5m`; AC 100 Hz – 150 MHz, 20 points/decade; DC sweep enabled.
5. Run **Simulation → Netlist and Run**.
6. Plot the inputs and `OUT` (transient), AC magnitude, and `dB20(VF("/out"))`. Expected results are in Figures 3 and 4.

### Step 4: Layout, DRC, LVS, extraction (optional)

Open `diff_amp` → `layout` (compare with Figure 5). These runs need the UMC 180 nm decks.

1. **DRC:** run Assura DRC; expect "no DRC errors found".
2. **LVS:** run Assura LVS against the schematic; expect a match with zero mismatches.
3. **Extraction:** run Quantus QRC on the LVS run, RC-decoupled, ground net `VSS`, view name `av_extracted`.

The extracted view is already included, so this step can be skipped for resimulation only.

### Step 5: Post-layout simulation

1. Open `temp_test` → `config`, or create a config whose top cell is the testbench with the binding:

   ```text
   inst (TTRP_RaviRupesh_kartik.temp_test:schematic).I0 binding :av_extracted;
   ```

2. Open the testbench schematic **through that config**.
3. Launch ADE L and run the same transient, AC and DC analyses.
4. Plot `OUT` and `dB20(VF("/out"))`; the passband marker should read about **−40.2 dB** (Figure 13).

### Reproduction checklist

- [ ] Schematic shows an NMOS pair, PMOS mirror and NMOS tail
- [ ] Pre-layout transient gives a sine at `OUT` for a differential sine input
- [ ] AC gain is flat at low frequency, then rolls off
- [ ] DRC reports no errors; LVS reports a match
- [ ] Post-layout run of `av_extracted` shows the same sine and a marker near −40.2 dB

---

## 12. Project structure

```text
Ravi_diff/
├── README.md
├── Image/                          figures referenced above
│   ├── Diff_amp_design_step1.png
│   ├── Pre_layout_sim_start_part1.png
│   ├── Prelayout_sim_plot1.png
│   ├── Pre_layout_sim_plot2.png
│   ├── Diff_amp_Layout.png
│   ├── Diff_amp_DRC_check.png
│   ├── Diff_amp_LVS_check.png
│   ├── Diff_amp_RC_extraction.png
│   ├── Post_layout_step1.png
│   ├── Post_layout_step2.png
│   ├── Post_layout_step3.png
│   ├── Post_layout_plot1.png
│   └── post_layout_plot2.png
├── diff_amp/                       Virtuoso cell
│   ├── schematic/
│   ├── symbol/
│   ├── layout/
│   ├── av_extracted/
│   ├── spectre_state1/
│   └── spectreText/
└── temp_test/                      testbench and post-layout config
    ├── schematic/
    └── config/
```
