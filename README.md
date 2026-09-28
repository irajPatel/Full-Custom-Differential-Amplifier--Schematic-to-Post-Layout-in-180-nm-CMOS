# Differential Amplifier


A single-ended differential amplifier designed, simulated, laid out, and verified in Cadence Virtuoso on the **UMC 180 nm** CMOS process.

This work was completed as part of the **TTRP Project**.

**Team:** Peaky Balwindars

| Team member |
| --- |
| Ravi Patel |
| Kartik Jhangir |
| Rupesh |
| Suryanshu Raghav |

**Library:** `TTRP_RaviRupesh_kartik`  
**Cell:** `diff_amp`

The pages below follow one path: why this circuit exists, how the transistor pair produces a voltage that depends on the *difference* of two inputs, how that idea was drawn in our schematic, and how the same circuit was checked from pre-layout simulation through layout, DRC, LVS, and post-layout simulation.

---

## 1. What is a differential amplifier?

A differential amplifier is a circuit with **two inputs** and an output that depends on the **difference** between those inputs.

If the two input voltages are \(v_{G1}\) and \(v_{G2}\), the amplifier is built to respond to

\[
v_{id} = v_{G1} - v_{G2}
\]

and to stay quiet when both inputs move up or down together.

In this project the output is **single-ended**: one wire, `OUT`, measured with respect to ground. Many later analog blocks (operational amplifiers, comparators, sense amplifiers) start from this same pair of transistors. Learning this cell is learning the front end of those blocks.

On our schematic the two inputs are the pins **A** and **B**. They are the gates of the two NMOS transistors that form the differential pair. The drains of that pair meet a PMOS current mirror, and the mirrored node is `OUT`.

---

## 2. Why do we need it?

A single-transistor amplifier treats every movement of its input as a signal. That includes the signal we care about, and it also includes noise, supply ripple, and interference that land on the input in the same way.

A differential amplifier gives us a way to separate those two kinds of motion.

| What happens at the inputs | What we want the output to do |
| --- | --- |
| The two voltages move in opposite directions | Produce a large output. This is the real signal. |
| The two voltages move in the same direction | Produce almost no output. This is interference that both wires picked up together. |

That second case is called **common mode**. A microphone cable, a sensor pair, or two long traces on a chip often pick up the same hum on both wires. The difference between the wires is still the message. The differential amplifier keeps the message and rejects the hum.

It is also the only practical way to subtract two voltages with transistors. An op-amp does \(v_{out} = A(v_+ - v_-)\). The circuit that actually forms \((v_+ - v_-)\) is a differential pair.

---

## 3. Basic working principle

Our cell has three ideas stacked on top of each other.

1. **A differential pair.** Two matched NMOS transistors, M1 and M2, share one source node. Their gates are the two inputs.
2. **A tail current source.** An NMOS transistor under that shared source sinks a fixed current \(I_{SS}\). The two input transistors must share this current. Whatever one of them takes, the other must give up.
3. **A PMOS current mirror as the load.** The current of one input transistor is copied to the output branch. The copy is compared with the current of the other input transistor. The difference becomes the output current, and that current, flowing through the output resistance, becomes \(v_{OUT}\).

A current mirror is a pair of transistors that share a gate voltage, so one of them is forced to carry a copy of the current in the other. Here the mirror is made of PMOS devices sitting between the input drains and VDD.

The supply is the core voltage of this process, **1.8 V**, between `VDD` and `VSS`.

---

## 4. How the differential pair responds to two inputs

It helps to split any pair of inputs into two pieces we can study separately:

\[
v_{ic} = \frac{v_{G1} + v_{G2}}{2}
\qquad
v_{id} = v_{G1} - v_{G2}
\]

\(v_{ic}\) is the **common-mode** voltage: the average level of the two inputs.  
\(v_{id}\) is the **differential-mode** voltage: the difference we want to amplify.

The same split written the other way is

\[
v_{G1} = v_{ic} + \frac{v_{id}}{2}
\qquad
v_{G2} = v_{ic} - \frac{v_{id}}{2}
\]

So each gate sees the average, plus or minus half of the difference.

### Quiescent point

With \(v_{id} = 0\) the two gates sit at the same voltage. The two NMOS transistors are drawn with the same width and the same length, so they match. They split the tail current into two equal parts:

\[
I_{D1} = I_{D2} = \frac{I_{SS}}{2}
\]

Nothing unbalanced is left to send to the output. \(v_{OUT}\) sits at its bias point.

### A pure differential input

Now let the common mode stay fixed and let the difference grow.

- The gate of M1 rises by \(v_{id}/2\).
- The gate of M2 falls by \(v_{id}/2\).

M1 wants more current. M2 wants less. The tail still supplies only \(I_{SS}\), so the extra current in M1 is exactly the current that M2 gave up.

For a small difference the change in each drain current is set by the **transconductance** \(g_m\). Transconductance is the small-signal gain of a MOS transistor from gate-source voltage to drain current, in siemens (A/V):

\[
i_{d1} = +\,g_m \frac{v_{id}}{2}
\qquad
i_{d2} = -\,g_m \frac{v_{id}}{2}
\]

The shared source node does not move for this pure differential case. The two source currents change by equal and opposite amounts, so they cancel at the tail node. That node is a **virtual ground** for differential signals. Each transistor therefore sees the full half-difference as its own \(v_{gs}\).

### Where the output current comes from

M1's current increase is copied by the PMOS mirror into the output branch. M2's current decrease means the output NMOS is sinking *less* than the mirror is sourcing. Both effects push current out of the output node, and they add:

\[
i_{out}
  = \left(\frac{I_{SS}}{2} + g_m\frac{v_{id}}{2}\right)
  - \left(\frac{I_{SS}}{2} - g_m\frac{v_{id}}{2}\right)
  = g_m\, v_{id}
\]

The factor of \(1/2\) disappeared because the mirror put the left-hand signal back onto the right-hand node. That is why a current-mirror load has about twice the gain of a single-ended pair that uses a plain resistor on only one side.

This \(i_{out}\) flows through the resistance seen at `OUT` and becomes the output voltage. That step is the gain formula in [Section 7](#7-the-equations-and-where-they-come-from).

---

## 5. Role of the tail current source

The tail is the NMOS transistor whose drain is the shared source of M1 and M2 and whose source sits on `VSS`. Its job is to hold \(I_{SS}\) steady.

Two things depend on that.

**It sets the bias current.** Each input transistor is biased at \(I_{SS}/2\). That current sets \(g_m\), and \(g_m\) sets the gain. In our testbench this current is fixed by a **20 µA** reference.

**It rejects the common mode.** If both gates rise together, the shared source rises with them. The gate-source voltage of M1 and M2 barely changes, so their currents barely change. The tail is what makes the source able to follow. An ideal tail has infinite output resistance and common-mode gain falls to zero. A real MOS tail has a large but finite output resistance, so a little common-mode current remains. The ratio of the gain we want to the gain we do not want is the CMRR, defined in [Section 8](#8-voltage-gain-and-the-other-parameters-that-matter).

The tail device on our schematic is **M0**, an `N_18_MM` transistor with \(W = 3.6\,\mu\mathrm{m}\) and \(L = 360\,\mathrm{nm}\). A second NMOS forms the bias branch that sets M0's gate from the reference current. Parasitic extraction of the finished layout counts **four** `N_18_MM` devices and **two** `P_18_MM` devices, which is this pair, this tail, the bias device, and the two PMOS load transistors.

---

## 6. Common-mode and differential-mode operation

Both modes are present every time the circuit is on. We look at them one at a time so each one stays clear.

### Differential mode

The inputs move in opposite directions. The tail current sloshes from one side of the pair to the other. The mirror turns that current difference into \(i_{out} = g_m v_{id}\), and `OUT` moves.

This is the mode we design for. The pre-layout and post-layout transient plots are this test: two sines of equal size, opposite phase, and an output sine at the same frequency.

### Common mode

The inputs move in the same direction. The pair stays balanced, each side still carries about \(I_{SS}/2\), and the mirror still sees two equal currents. The current into the output capacitor and the load is almost unchanged, so \(v_{OUT}\) almost stays still.

### What "in saturation" means here

A MOS transistor is in **saturation** when its drain-source voltage is large enough that the channel is pinched off and the drain current is set by the gate-source voltage. That is the region in which \(i_d = g_m v_{gs}\) is a fair description.

The amplifier does its job while every transistor in the signal path stays in saturation:

- M1 and M2 must have enough \(V_{DS}\) above the tail.
- The PMOS mirror devices must have enough \(V_{SD}\) below VDD.
- The tail must stay in saturation so \(I_{SS}\) does not wander.

The range of \(v_{ic}\) over which this remains true is the **input common-mode range**. The DC sweep in the post-layout section is the picture of that story: inside a window of input voltage the output moves steeply, and outside that window one side of the pair takes all of the tail current and the output flattens.

---

## 7. The equations, and where they come from

Every symbol below is a voltage or a current on the schematic you can point at.

| Symbol | Meaning on this circuit |
| --- | --- |
| \(v_{G1}, v_{G2}\) | Gate voltages of the two input NMOS transistors (pins A and B) |
| \(v_{id}\) | \(v_{G1} - v_{G2}\), the differential input |
| \(v_{ic}\) | Average of the two gate voltages |
| \(I_{SS}\) | Current sunk by the tail, M0 |
| \(I_{D1}, I_{D2}\) | Drain currents of the two input transistors |
| \(g_m\) | Transconductance of **one** input transistor, biased at \(I_{SS}/2\) |
| \(r_{o2}, r_{o4}\) | Small-signal output resistances of the output NMOS and the output PMOS |
| \(v_{OUT}\) | Voltage at the single-ended output pin |

### 7.1 Tail current splits between the two sides

KCL at the shared source node:

\[
I_{D1} + I_{D2} = I_{SS}
\]

This is why the pair can turn a voltage difference into a current difference. The sum is fixed. Only the split can change.

### 7.2 Small-signal current in each input transistor

In saturation the drain current of a MOS transistor depends on its overdrive. **Overdrive** \(V_{OV}\) is the amount by which the gate-source voltage exceeds the threshold voltage:

\[
V_{OV} = V_{GS} - V_{TH}
\]

Differentiating the square-law drain current with respect to \(V_{GS}\) gives

\[
g_m = \mu_n C_{ox} \left(\frac{W}{L}\right) V_{OV} = \frac{2 I_D}{V_{OV}} = \sqrt{2\,\mu_n C_{ox}\,\frac{W}{L}\,I_D}
\]

For M1 and M2 the current in this formula is \(I_D = I_{SS}/2\), and \(W/L\) is the size printed on those devices in the schematic. A wider device, or more bias current, raises \(g_m\). A larger overdrive lowers \(g_m\) for the same current. The three forms are the same number, written so we can use whichever bias data we have.

Because the source is a virtual ground for a differential input,

\[
i_{d1} = g_m \frac{v_{id}}{2}, \qquad i_{d2} = -g_m \frac{v_{id}}{2}.
\]

### 7.3 Why the mirror makes \(i_{out} = g_m v_{id}\)

The diode-connected PMOS on the left copies M1's current into the PMOS on the right. Subtracting M2's current from that copy, as written in [Section 4](#where-the-output-current-comes-from), leaves

\[
i_{out} = g_m\, v_{id}.
\]

### 7.4 Output voltage and voltage gain

The output node is not a short. Looking into it we see the output resistance of the NMOS (M2) in parallel with the output resistance of the PMOS that feeds it:

\[
R_{out} = r_{o2} \parallel r_{o4}
\]

A MOS transistor's \(r_o\) is the small slope of \(I_D\) versus \(V_{DS}\) in saturation. It is large when the channel is long and the current is small. Our PMOS loads are drawn at \(L = 720\,\mathrm{nm}\), twice the input-device length, which raises their \(r_o\) and therefore raises \(R_{out}\).

Then

\[
v_{out} = i_{out}\, R_{out} = g_m R_{out}\, v_{id}
\]

and the differential voltage gain is

\[
A_{dm} = \frac{v_{out}}{v_{id}} = g_m \left(r_{o2} \parallel r_{o4}\right).
\]

This is the number the AC analysis is measuring. In dB,

\[
A_{dm,\mathrm{dB}} = 20 \log_{10} |A_{dm}|.
\]

In ADE the curve we plotted is `dB20(VF("/out"))`, which is \(20\log_{10}|v_{OUT}|\). With an AC input magnitude of 1 V, that curve **is** \(A_{dm}\) in dB. A reading of \(-40\,\mathrm{dB}\) means \(|A_{dm}| \approx 0.01\). The sign is the logarithm of a number smaller than 1. It is the gain at the bias point, which is also the local slope of the DC transfer curve at that same bias point.

### 7.5 Common-mode gain and CMRR

For a pure common-mode input, \(v_{id} = 0\) and both gates move by \(v_{ic}\). The source follows, so \(v_{gs}\) of each input device changes only because the tail has a finite resistance. The leftover output is

\[
A_{cm} = \frac{v_{out}}{v_{ic}}.
\]

**CMRR** (common-mode rejection ratio) says how many times larger the wanted gain is than this leftover:

\[
\mathrm{CMRR} = \left|\frac{A_{dm}}{A_{cm}}\right|
\qquad
\mathrm{CMRR}_{\mathrm{dB}} = 20\log_{10}\left|\frac{A_{dm}}{A_{cm}}\right|.
\]

A stiffer tail (higher output resistance, longer channel, or a cascode) raises CMRR. Matching of M1 and M2 matters just as much: if the two transistors are not equal, a common-mode voltage turns into a false difference.

### 7.6 A single pole sets the bandwidth

\(R_{out}\) is large, and `OUT` always has some capacitance: the drain junctions, the mirror gate, and, after layout, the wire capacitance. Together they form one time constant

\[
\omega_p \approx \frac{1}{R_{out} C_{out}}, \qquad f_p \approx \frac{1}{2\pi R_{out} C_{out}}.
\]

Below \(f_p\) the gain is flat. Above \(f_p\) it falls. That is the shape of every AC plot in this report. Layout adds capacitance that the schematic netlist does not contain, so \(f_p\) is expected to move when we resimulate the extracted view. That is the reason the post-layout run exists.

---

## 8. Voltage gain and the other parameters that matter

| Parameter | What it tells us | How this design sets it |
| --- | --- | --- |
| \(A_{dm} = g_m R_{out}\) | How many volts of output we get per volt of difference | \(g_m\) from the input size and \(I_{SS}/2\); \(R_{out}\) from the NMOS and PMOS at `OUT` |
| \(g_m = 2 I_D / V_{OV}\) | How strongly each input transistor converts voltage into current | \(I_D = I_{SS}/2\), with \(I_{SS}\) set by the 20 µA bias |
| \(R_{out} = r_{o2}\parallel r_{o4}\) | How large a voltage \(i_{out}\) can build | PMOS length is 720 nm so the load stays a high resistance |
| CMRR | How well a shared disturbance is ignored | Set by tail resistance and by M1–M2 matching |
| Input common-mode range | The range of average input voltage for which every device stays in saturation | Must sit high enough to keep the tail in saturation and low enough to keep the pair in saturation |
| Output swing | How far `OUT` can move before a device leaves saturation | From one overdrive above the tail node up to one overdrive below VDD |
| Bandwidth \(f_p\) | Where the flat gain starts to fall | Set by \(R_{out} C_{out}\); confirmed on the AC plots |
| Power | Supply current times 1.8 V | The signal branch draws \(I_{SS}\). The bias branch draws the reference current as well |

Power is easy to estimate from the testbench. The reference source is 20 µA. Mirrored 1:1 into the tail, the cell draws that 20 µA again in the signal branch, so the supply current is about 40 µA and the power is about

\[
P \approx 1.8\,\mathrm{V} \times 40\,\mu\mathrm{A} = 72\,\mu\mathrm{W}.
\]

That estimate counts the bias branch and the signal branch. It is a bias calculation, separate from the gain read off the AC plot.

---

## 9. Design specifications

The amplifier was designed for the UMC 180 nm core devices and checked at the bias point below.

| Item | Value |
| --- | --- |
| Process | UMC 180 nm CMOS (`UMC_18_CMOS`) |
| NMOS / PMOS models | `N_18_MM` / `P_18_MM` |
| Supply | 1.8 V |
| Temperature | 27 °C |
| Tail bias | 20 µA reference in the testbench |
| Input pair M1, M2 | NMOS, \(W = 1.44\,\mu\mathrm{m}\), \(L = 360\,\mathrm{nm}\) |
| PMOS current-mirror load | PMOS, \(W = 1.44\,\mu\mathrm{m}\), \(L = 720\,\mathrm{nm}\) |
| Tail device M0 | NMOS, \(W = 3.6\,\mu\mathrm{m}\), \(L = 360\,\mathrm{nm}\) |
| Output style | Single-ended (`OUT`) |
| Simulator | Cadence Spectre, ADE L |
| Layout checks | Assura DRC and Assura LVS |
| Parasitic extraction | Quantus QRC, view `av_extracted` |

The three analyses used for every electrical check:

| Analysis | Setting | What we learn |
| --- | --- | --- |
| Transient | Stop time 5 ms | The output waveform for a small differential sine |
| AC | 100 Hz to 150 MHz, 20 points per decade | Small-signal gain and the frequency where it rolls off |
| DC | Sweep of one input across the supply | The large-signal transfer curve and the bias region |

---

## 10. Schematic design

The cell `diff_amp` was entered in Virtuoso Schematic Editor. The screenshot below is the design.

![Transistor schematic of diff_amp](Image/Diff_amp_design_step1.png)

*Figure 1. Schematic of `diff_amp`. The NMOS pair sits in the middle, the PMOS mirror is on top toward `VDD`, and the tail device returns to `VSS`. Pins A and B are the inputs. `OUT` leaves from the mirror output.*

**What is shown.** Each transistor is an instance of the UMC 180 nm device (`N_18_MM` or `P_18_MM`) with the width and length printed on the symbol. The property editor in the same window shows the tail device M0: total width 3.6 µm, length 360 nm, one finger.

**What we checked.** The two input devices match each other. The two PMOS load devices match each other. The tail is in series with the shared source. `OUT` is taken from the side of the mirror that is not diode-connected. Supplies are `VDD` and `VSS`.

**Why this stage exists.** Every later step netlists this drawing. A wrong connection here is invisible in the layout plots and shows up as a surprising waveform. The schematic is the specification the layout has to match.

Device sizes read from this schematic:

| Device | Role | Model | Width | Length |
| --- | --- | --- | --- | --- |
| M1, M2 | Input pair | `N_18_MM` | 1.44 µm | 360 nm |
| PMOS mirror | Active load | `P_18_MM` | 1.44 µm | 720 nm |
| M0 | Tail current source | `N_18_MM` | 3.6 µm | 360 nm |

---

## 11. Circuit working

Follow one differential half-cycle on Figure 1.

1. Pin A rises and pin B falls by the same small amount. The common mode stays near 0.9 V, which is the center of the input waveforms in the transient plots.
2. The left input NMOS takes more of the tail current. The right input NMOS takes less. The sum is still the tail current.
3. The diode-connected PMOS on the left carries that larger current, and its gate voltage moves with it.
4. The other PMOS, which shares that gate, copies the larger current into the output branch.
5. The right input NMOS is sinking a smaller current than the PMOS is sourcing, so the extra current charges the output node and \(v_{OUT}\) rises.

On the opposite half-cycle the whole story reverses and \(v_{OUT}\) falls. A sine in, a sine out, at the same frequency. That is the waveform the transient analysis has to show before we trust the gain number.

The testbench wrapped around the symbol is simple on purpose.

| Source | Value | Purpose |
| --- | --- | --- |
| `vdc` | 1.8 V | Core supply |
| `idc` | 20 µA | Reference current that biases the tail |
| `vsin` | 5 mV amplitude, common mode near 0.9 V | A small differential sine, so the response stays in the linear region |

A small amplitude matters. The gain equation \(A_{dm} = g_m R_{out}\) is a small-signal result. A large input would slam the output into the supply rails and the measured ratio would no longer equal \(g_m R_{out}\).

---

## 12. Pre-layout simulation

Pre-layout simulation runs the schematic netlist only. Wires are ideal. This is the electrical design check, done before any polygon is drawn.

The state is saved in the cell as `spectre_state1`. The ADE window below is that setup: transient, AC, and DC enabled together, Spectre as the simulator, temperature 27 °C.

![ADE L testbench before layout](Image/Pre_layout_sim_start_part1.png)

*Figure 2. Pre-layout testbench. The `diff_amp` symbol is biased from 1.8 V and 20 µA, with a 5 mV sine on the input. ADE L is set to run transient, AC, and DC.*

**What we checked.** The netlist compiles, the bias point exists, and all three analyses finish. Outputs saved for plotting are the two input nets and `/out`.

**Why this stage exists.** There is no point drawing layout for a schematic that does not amplify. Pre-layout is also the reference we compare against after parasitics are added.

### Transient and AC magnitude

![Pre-layout transient and AC magnitude](Image/Prelayout_sim_plot1.png)

*Figure 3. Pre-layout results. Left: transient inputs and output over 5 ms. Right: AC magnitude of the same nets from 100 Hz to 150 MHz.*

**What the transient shows.** The two inputs are sines centered near 0.9 V, equal in size and opposite in phase. That is \(v_{id}\) with a fixed \(v_{ic}\). `OUT` is a sine at the same frequency, which is what Section 4 predicts when the pair steers the tail current back and forth.

**What the AC magnitude shows.** The input magnitudes stay flat, because the sources are ideal. The output magnitude stays flat at low frequency and then falls. That flat region is \(A_{dm}\). The fall is the single pole \(1/(R_{out} C_{out})\) from Section 7.6. At this stage \(C_{out}\) is only the device capacitance inside the schematic.

### Gain in decibels

![Pre-layout gain in dB](Image/Pre_layout_sim_plot2.png)

*Figure 4. The same pre-layout AC run plotted as `dB20(VF("/out"))`, next to the magnitude plot. The passband is flat, then the gain rolls off.*

**What we checked.** The dB curve has a clear passband and a clean roll-off, with no peaking. Peaking would mean a second pole interacting with the output pole. A single smooth roll-off matches the one-pole model.

**Why we plot dB.** A linear magnitude plot squashes the passband and the roll-off onto very different scales. Decibels turn the same data into a curve we can read: flat, then falling.

---

## 13. Layout design

The same cell was drawn in Virtuoso Layout Suite XL in the UMC 180 nm layers. Schematic names were carried onto the layout so the nets we simulated (`VDD`, `VSS`, A, B, `OUT`) are the nets on the silicon.

![Layout of diff_amp](Image/Diff_amp_Layout.png)

*Figure 5. Layout of `diff_amp` in the UMC 180 nm layers. The two wide devices along the top are the PMOS loads. The input pair sits in the middle. The tail devices sit along the bottom. Metal connects `VDD`, `VSS`, the inputs, and `OUT`.*

**What is shown.** Diffusion, poly, contacts, and Metal1. Supply rails run as metal straps so the transistors see a real `VDD` and `VSS`. The input pair is placed as a matched couple, which is what keeps \(I_{D1}\) and \(I_{D2}\) equal when \(v_{id} = 0\).

**What we checked.** Every schematic device has a layout device. Inputs, output, and supplies are labeled on the metal that actually carries them. The pair and the mirror are placed so the matched transistors see a similar neighborhood.

**Why this stage exists.** The schematic has no width of wire, no sidewall capacitance, and no contact resistance. The layout is the circuit that would be fabricated. DRC, LVS, and extraction all read this view.

---

## 14. DRC verification

DRC (design-rule check) asks whether every polygon obeys the foundry's spacing, width, and enclosure rules. A circuit that is electrically perfect and violates a rule will not manufacture reliably.

Assura was run on the `diff_amp` layout with the UMC 180 nm rule deck.

![DRC result: no errors](Image/Diff_amp_DRC_check.png)

*Figure 6. Assura DRC on the `diff_amp` layout. The dialog reports "No DRC errors found."*

**What we checked.** Width and spacing of diffusion, poly, contacts, and metal; enclosure of contacts; and the rest of the UMC 180 nm deck loaded for this process.

**Why this stage exists.** LVS can pass on a layout that still has a rule violation. DRC is the manufacturing check, and it has to be clean before we trust the extracted netlist. The result here is a clean deck: **no DRC errors**.

---

## 15. LVS verification

LVS (layout versus schematic) extracts the devices and connections from the polygons and compares them with the schematic netlist. It answers a different question from DRC: not "is it legal?" but "is it the same circuit?"

Assura LVS was run as run `exp1` on library `TTRP_RaviRupesh_kartik`, cell `diff_amp`.

![LVS result: schematic and layout match](Image/Diff_amp_LVS_check.png)

*Figure 7. Assura LVS summary. Schematic and layout match. The comparison lists zero net mismatches, zero device mismatches, zero pin mismatches, and zero parameter mismatches. Total DRC violations reported with the run: 0.*

**What we checked.**

| LVS item | Result |
| --- | --- |
| Schematic and layout | Match |
| Malformed devices, shorts, opens | None |
| Net mismatches | 0 |
| Device mismatches | 0 |
| Pin mismatches | 0 |
| Parameter mismatches | 0 |
| DRC violations reported with the run | 0 |

**Why this stage exists.** A layout can be DRC-clean and still swap a drain and a source, drop a mirror connection, or draw a width that is not the schematic width. Any of those would make the post-layout simulation a simulation of a different circuit. A match is the permission to extract parasitics and resimulate.

---

## 16. Post-layout simulation

Post-layout simulation is the schematic connectivity plus the resistance and capacitance of the wires we actually drew.

### Extraction

Quantus QRC (run `exp1`) built an extracted cellview named `av_extracted`. The extraction is RC-decoupled, with `VSS` as the ground net, using the UMC 180 nm LPE deck. The run finished normally and wrote the view back into `diff_amp`.

![Quantus extraction completed](Image/Diff_amp_RC_extraction.png)

*Figure 8. Quantus QRC finished and created `diff_amp` / `av_extracted`. The log shows the device count of the extracted cell.*

The extracted view contains the intended transistors plus the parasitic elements Quantus inserted for the routing:

| Extracted instance | Count | What it is |
| --- | --- | --- |
| `N_18_MM` | 4 | Input pair, tail, and bias NMOS |
| `P_18_MM` | 2 | PMOS current-mirror load |
| `presistor` | 51 | Parasitic wire and contact resistance |
| `pcapacitor` | 35 | Parasitic capacitance to the ground net |

Those 51 resistors and 35 capacitors are absent from the pre-layout netlist. They are the reason this section exists.

### Pointing the testbench at the extracted view

The testbench schematic still contains a symbol of `diff_amp`. A **config** view tells Virtuoso to use `av_extracted` for that instance and to keep the ideal sources as schematic devices. The cell `temp_test` in this project holds that configuration: instance `I0` is bound to `av_extracted`.

The three windows below are the same operation in the GUI.

![Opening the testbench cell](Image/Post_layout_step1.png)

*Figure 9. Library Manager, with the testbench cell open for the post-layout run.*

![New config view](Image/Post_layout_step2.png)

*Figure 10. Creating the config view that will decide which representation of `diff_amp` is simulated.*

![Config bound to the extracted view](Image/Post_layout_step3.png)

*Figure 11. Hierarchy Editor binding `diff_amp` to `av_extracted`, and the testbench schematic opened through that config. ADE still runs Spectre on the same sources: 1.8 V, 20 µA, and the input sine.*

**What we checked.** The config lists `av_extracted` as the view to use for our cell, while `analogLib` sources stay on their Spectre views. The netlist that ADE runs is therefore the extracted netlist.

**Why the config is required.** Netlisting the bare schematic again would repeat Section 12 and would never see the 51 resistors or the 35 capacitors.

### Post-layout waveforms

![Post-layout transient, AC, and DC](Image/Post_layout_plot1.png)

*Figure 12. Post-layout results on the extracted view. Top left: transient. Top right: AC magnitude. Bottom left: DC transfer curve.*

**Transient.** The two inputs are again opposite-phase sines around a common mode near 0.9 V, and `OUT` is a sine at the same frequency. The layout still implements the differential pair. The wiring did not break the steering action derived in Section 4.

**AC magnitude.** Input magnitudes stay flat. The output magnitude is flat at low frequency and then falls, the same single-pole shape as the pre-layout run, now with the extracted \(C_{out}\) included.

**DC sweep.** One input is held near the common-mode level and the other is swept across the supply. `OUT` moves through a steep region and then flattens. The steep region is where both sides of the pair are in saturation and \(A_{dm} = g_m R_{out}\) applies. The flat regions are where one transistor has taken all of \(I_{SS}\) and the small-signal model no longer describes the circuit. This plot is how we see the input range discussed in Section 6.

### Gain after layout

![Post-layout gain with marker](Image/post_layout_plot2.png)

*Figure 13. Post-layout AC gain, `dB20(VF("/out"))`. A marker on the passband reads **−40.196 dB**.*

**What the number means.** \(-40.196\,\mathrm{dB}\) is \(20\log_{10}|v_{OUT}|\) at the marker. With a 1 V AC stimulus this is the small-signal voltage gain at the bias point:

\[
|A_{dm}| = 10^{-40.196/20} \approx 0.0098.
\]

The curve is flat through the low-frequency decades and then falls, so the cell behaves as the single-pole amplifier of Section 7. The marker is the gain we quote for the finished layout, because it includes the wiring.

---

## 17. Comparison of pre-layout and post-layout results

Both runs use the same testbench, the same three analyses, and the same definition of gain. The only intentional change is the view under the symbol: `schematic` before layout, `av_extracted` after it.

| Check | Pre-layout | Post-layout |
| --- | --- | --- |
| Netlist | Schematic devices only | Schematic devices plus 51 parasitic resistors and 35 parasitic capacitors |
| Transient | Opposite-phase input sines, sinusoidal `OUT` | Same differential behavior on the extracted view |
| AC shape | Flat passband, then a single roll-off | Flat passband, then a single roll-off |
| Gain curve | `dB20(VF("/out"))` flat at low frequency, then falling | Passband marker at **−40.196 dB**, then the same roll-off |
| DC transfer | Large-signal sweep of one input | Steep region around the bias point, flat outside it |
| What can move the result | Device capacitance already inside the models | Added wire capacitance and resistance |

The shapes agree, which is what LVS led us to expect: the layout is the schematic. The numbers are allowed to shift. Extra \(C_{out}\) moves the pole \(f_p \approx 1/(2\pi R_{out} C_{out})\) down, and a small series resistance with the output devices trims \(R_{out}\) and therefore \(A_{dm}\). The post-layout marker is the gain that belongs to the drawn cell.

DRC-clean and LVS-clean results are what make this comparison meaningful. Because LVS matched, a shift between the two gain curves comes from the extracted resistance and capacitance.

---

## 18. Final results

| Result | Value |
| --- | --- |
| Process and supply | UMC 180 nm, 1.8 V |
| Topology | NMOS differential pair, NMOS tail, PMOS current-mirror load, single-ended output |
| Input pair | \(1.44\,\mu\mathrm{m}\,/\,360\,\mathrm{nm}\) |
| PMOS load | \(1.44\,\mu\mathrm{m}\,/\,720\,\mathrm{nm}\) |
| Tail (M0) | \(3.6\,\mu\mathrm{m}\,/\,360\,\mathrm{nm}\) |
| Devices in the extracted cell | 4 × `N_18_MM`, 2 × `P_18_MM` |
| Bias | 20 µA reference, 27 °C |
| Estimated bias power | about 72 µW, counting the reference branch and the tail |
| DRC | No errors |
| LVS | Schematic and layout match, 0 mismatches |
| Extraction | `av_extracted`, RC-decoupled, Quantus run `exp1` |
| Post-layout small-signal gain | **−40.196 dB** at the AC marker (\(\|A_{dm}\| \approx 0.0098\)) |
| Frequency response | Low-frequency passband, then a single-pole roll-off |

### Conclusion

The differential amplifier does what the first half of this document derived. Two matched NMOS transistors share a tail current, so a difference between their gates becomes a difference in their drain currents. The PMOS mirror folds that difference onto one node, and \(v_{OUT} = g_m R_{out}\, v_{id}\) while the transistors stay in saturation.

On this project that circuit was carried all the way through a 180 nm design cycle. The schematic sizes are the ones in Figure 1. Pre-layout Spectre runs showed a sinusoidal differential response and a one-pole AC curve. The layout passed DRC with no errors and matched the schematic in LVS with no mismatches. Quantus wrote the wiring back as `av_extracted`, and the post-layout AC marker on that view reads **−40.196 dB**. The post-layout transient still shows a sine at `OUT` for a differential sine at the inputs, which is the same mechanism, now with the layout parasitics included.

---

## 19. How to reproduce this design

The cells in this folder are Cadence OpenAccess libraries. Another person can reopen them in Virtuoso, rerun Spectre, and walk the same DRC, LVS, and extraction path.

### What you need

- Cadence Virtuoso IC 6.1.8 (the database was saved with this stream; a nearby IC 6.1 release will open it)
- Spectre, for the electrical runs
- The UMC 180 nm design kit already used in the lab: library `UMC_18_CMOS`, Assura rule decks, and the Quantus LPE deck
- The Cadence `analogLib` and `basic` reference libraries

### Files in this project

| Path | What it is |
| --- | --- |
| `diff_amp/schematic` | Transistor schematic |
| `diff_amp/symbol` | Symbol used by the testbench |
| `diff_amp/layout` | Layout that was DRC- and LVS-checked |
| `diff_amp/av_extracted` | Quantus RC-extracted view |
| `diff_amp/spectre_state1` | Saved ADE state (transient, AC, DC, outputs, model setup) |
| `temp_test/schematic` | Testbench schematic that instances `diff_amp` |
| `temp_test/config` | Config that binds the instance to `av_extracted` |
| `Image/` | Figures used in this document |

Stale lock files (`*.cdslck`) came along from the lab session. If Virtuoso opens a cell read-only, close every session that might hold it and remove those lock files from that cell directory.

### 1. Tell Virtuoso where the library is

From the directory that contains your `cds.lib`, point the library name at this folder. The cells were saved under the library name `TTRP_RaviRupesh_kartik`.

```text
DEFINE TTRP_RaviRupesh_kartik  /path/to/Ravi_diff
DEFINE UMC_18_CMOS             /path/to/UMC180/UMC_18_CMOS
```

`analogLib` and `basic` should already be defined by the site `cds.lib`. The model files referenced by the saved ADE state live under the kit's `Models/Spectre` directory (the `mm180_reg18` typical section is the one this cell uses).

### 2. Start Virtuoso and open the schematic

```bash
virtuoso &
```

In the Library Manager:

1. Select library `TTRP_RaviRupesh_kartik`.
2. Select cell `diff_amp`.
3. Open the `schematic` view.

You should see the pair, the PMOS mirror, the tail, pins A and B, and `OUT`, as in Figure 1.

### 3. Run the pre-layout simulation

1. Open the testbench schematic (`temp_test`, or the `test_diff_amp` cell if it is present in your copy of the library).
2. Confirm the sources: supply 1.8 V, bias current 20 µA, input sine amplitude 5 mV, common mode near 0.9 V.
3. Launch **ADE L**.
4. Load the saved state `spectre_state1` (**Session → Load State**) if you want the same analyses and the same plotted nets (`/out` and the two input nets).
5. Check the analyses:
   - transient, stop time `5m`
   - AC from `100` Hz to `150M` Hz, logarithmic, 20 points per decade
   - DC sweep enabled
6. **Simulation → Netlist and Run**.
7. Plot the transient inputs and `OUT`, the AC magnitude, and `dB20(VF("/out"))`.

The shapes to expect are Figures 3 and 4: opposite-phase input sines, a sinusoidal output, and a gain curve that is flat and then rolls off.

### 4. Open the layout

In the Library Manager open `diff_amp` → `layout`. The view should match Figure 5.

### 5. Recreate DRC, LVS, and extraction if you need to

These runs need the UMC 180 nm Assura and Quantus decks from the lab kit.

1. **DRC.** In the layout window, run Assura DRC with the UMC 180 nm deck. A clean run reports that no DRC errors were found.
2. **LVS.** Run Assura LVS with the schematic of `diff_amp` against this layout. A clean run reports that schematic and layout match, with zero net, device, pin, and parameter mismatches.
3. **Extraction.** Run Quantus QRC on that LVS run (`exp1` in our session), RC-decoupled, ground net `VSS`, output view name `av_extracted`. A finished run writes `diff_amp/av_extracted`.

If you only want to resimulate, the extracted view is already in the project and steps 1–3 can be skipped.

### 6. Run the post-layout simulation

1. Open `temp_test` → `config`, or create a config whose top cell is the testbench and whose binding for `diff_amp` is the view `av_extracted`. The checked-in config already contains:

```text
inst (TTRP_RaviRupesh_kartik.temp_test:schematic).I0 binding :av_extracted;
```

2. Open the testbench schematic **through that config**.
3. Launch ADE L and run the same transient, AC, and DC analyses.
4. Plot `OUT` in transient, in AC magnitude, and as `dB20(VF("/out"))`.

The passband marker to look for on the gain curve is **−40.196 dB**, as in Figure 13.

### 7. What "the same result" means

You have reproduced the project when all of the following are true.

- The schematic opens as an NMOS pair with a PMOS mirror and an NMOS tail.
- Pre-layout transient shows a sine at `OUT` for a differential sine at the inputs.
- The AC gain is flat at low frequency and then rolls off.
- DRC reports no errors, and LVS reports a match.
- Post-layout simulation of `av_extracted` still shows that sine, and the low-frequency gain marker is about **−40.2 dB**.

---

## Project structure

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
