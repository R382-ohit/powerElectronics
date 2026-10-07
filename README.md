# Rectifier Lab

An interactive power electronics simulator for **single-phase and three-phase rectifiers** built with diodes or thyristors. Pick a circuit, change the load, and watch the schematic, current path, waveforms and results respond live.

**Author:** Rohit Kumar (24EE10083)

It is a single file (`index.html`) with no build step and no dependencies. Google Fonts is loaded if online, with system fonts as fallback.

---

## 1. Circuits covered

| Supply | Type | Circuit | Devices |
|---|---|---|---|
| Single-phase | Half-wave | One device in series with the load | D1 / T1 |
| Single-phase | Full-wave | Full bridge | D1–D4 / T1–T4 |
| Three-phase | Half-wave | 3-pulse (M3) | D1–D3 / T1–T3 |
| Three-phase | Full-wave | 6-pulse bridge | D1–D6 / T1–T6 |

Each can be built with **diodes** (uncontrolled) or **thyristors** (controlled, firing angle α), with an optional **freewheeling diode (DF)** across the load, and with an **R, R–L or R–L–E** load.

---

## 2. Features

### Circuit builder (left menu)
- Open with the **☰ hamburger** button or the `M` key. Press `M` again (or `Esc`, ✕, or click outside) to close it.
- It asks one question at a time: **Supply → Rectifier type → Control → Freewheeling diode → Load**. Each answer opens the next step, and finished steps collapse to a one-line summary you can click to change.
- It opens by itself on your first visit only.
- The summary button above the sliders (for example "Single-phase · Half-wave · Controlled · Inductive") reopens it.

### Component values
Every value has a **slider (log scale)** and a **number box** where you can type an exact value.

| Value | Range | Default |
|---|---|---|
| Firing angle α (thyristor only) | 0 – 180° | 30° |
| Peak phase voltage Vm | 1 V – 100 kV | 325 V |
| Supply frequency f | 1 Hz – 2 kHz | 50 Hz |
| Resistance R | 1 mΩ – 1 MΩ | 10 Ω |
| Inductance L | 0 – 100 H (entered in mH) | 50 mH |
| Back-EMF E | 0 – 100 kV | 0 V |

**Preset chips** above the sliders: **R** (L = 0, E = 0), **R–L** (L = 50 mH, E = 0), **R–L–E** (L = 50 mH, E = 30 % of Vm) and **Reset** (all defaults). The active chip clears when you move a slider.

### Schematic
- The **conducting devices light up** and the rest stay plain.
- The **current path is animated** in the direction the current actually flows. In the bridges the source wires reverse between half-cycles.
- The load box shows the current R, L and E in engineering units. The freewheeling diode is drawn across the load when enabled, with its own current path.
- A **mode badge** (CCM / Boundary / DCM) sits in the panel header.

### Playback and readout
- **Play/Pause**, a **0–720° scrubber** (two cycles) and a speed selector (0.25×, 0.5×, 1×).
- The readout shows ωt, vo, io and one **pill per device**, lit while that device conducts, plus a short note on commutations and turn-on events.

### Waveforms
1. **Voltage waveforms:** source voltage(s) (va, or va/vb/vc) and the load voltage vo.
2. **Current waveforms:** phase-a source current ia and load current io. **Red shading** marks intervals where io = 0 (discontinuous conduction).
3. **Device conduction and gate pulses:** one row per device with bars while it conducts. For thyristors, pale bars show the gate pulse window starting at α. DF gets its own row.

### Cursor (linked across all views)
- **Drag** or **click** on any plot to move the cursor. It moves together on all three plots, and the schematic, current path and readout follow it.
- The cursor badge shows the angle and the value (vo on the voltage plot, io on the current plot, conducting devices on the device plot).
- With a plot focused, `←` / `→` move the cursor by 5°.

### Zoom
- **Right-click and drag** (or **Shift + drag**) to draw a box and zoom into it.
- The time (angle) zoom is **shared by all three plots**. Voltage and current plots also zoom vertically.
- **Double-click** a plot or press **Reset zoom** to return. The vertical zoom resets automatically when you change a circuit or value.

### Full screen
Each panel (schematic, voltage, current, device conduction) has a **⛶** button and opens full screen on its own. Press the button again (✕ Exit) or `Esc` to leave. The schematic's full screen view keeps the playback controls and readout.

### Results panel
**Vdc, Vrms (out), Idc, Irms (load), Ripple factor, Input PF, Mode, Theory Vdc.** Hover a tile for a one-line explanation. Values use engineering prefixes (µ, m, k, M).

### Appearance and shortcuts
- **Light / dark** theme switch (top right), remembered in your browser. It follows your system setting the first time.
- Shortcuts (when not typing in a field): `Space` play/pause · `←` `→` step the cursor · `T` theme · `M` circuit menu · `Esc` close.

---

## 3. How to use it

1. Open `index.html` (or the deployed page). If the menu opens, answer the steps, then press **Show simulation**.
2. Adjust R, L, E, Vm, f (and α for thyristors) with the sliders or type exact values.
3. Press **Play**, or drag the cursor on a waveform, to see which devices conduct and how the current flows.
4. Check the Mode badge and the results. Compare **Vdc** with **Theory Vdc**.
5. Right-drag on a plot to zoom into a commutation or a zero-current interval. Use ⛶ for a large view.

Things worth trying:
- Raise α on a thyristor bridge and watch Vdc fall, then go negative-going (inverter-like) near 90°.
- Increase L until the red zero-current shading disappears (discontinuous to continuous).
- Turn the freewheeling diode on for a half-wave circuit with an inductive load and watch it become continuous.
- Raise E toward the peak voltage and see conduction narrow, then stop when E exceeds the source.

---

## 4. The model and calculations

### 4.1 Simulation model
- **Ideal sources and devices:** zero on-state drop, instantaneous commutation, no source inductance.
- **Sources:** va = Vm sin ωt, vb = Vm sin(ωt − 120°), vc = Vm sin(ωt − 240°), with ω = 2πf.
- **Load equation:** L·di/dt = vo − E − R·i.
  Each step uses the exact update i ← iss + (i − iss)·e^(−R·Δt/L) with iss = (vo − E)/R, so it is stable for any R and L. For L = 0, i = iss.
- **Time stepping:** 1200 steps per cycle. The program starts from an estimated DC current and runs enough cycles to settle (up to 160). The last two cycles are plotted and all results come from the last cycle.
- **Conduction logic:** the circuit is a set of *paths*, each with a voltage vk and a firing angle.
  - A path conducts when its gate is on and vk is greater than the voltage of the path currently conducting (or greater than E when nothing conducts).
  - Thyristor gate pulses are **20° wide** starting at θk + α, where θk is the natural firing angle. Diodes are always gated (α = 0).
  - A path turns off when the current falls to zero.
- **Freewheeling diode:** if the conducting path voltage goes negative, DF takes over, vo is clamped to 0 and the thyristors turn off. DF stops when the current reaches zero. While DF conducts the source current is zero.
- When nothing conducts, vo = E (the load sees only its back-EMF).

### 4.2 Paths and devices per circuit

| Circuit | Paths (voltage → devices) | Natural firing angles θk |
|---|---|---|
| 1φ half-wave | va → D1 | 0° |
| 1φ full-wave bridge | +va → D1, D3  ·  −va → D2, D4 | 0°, 180° |
| 3φ half-wave (3-pulse) | va → D1 · vb → D2 · vc → D3 | 30°, 150°, 270° |
| 3φ bridge (6-pulse) | vab → D1, D6 · vac → D1, D2 · vbc → D3, D2 · vba → D3, D4 · vca → D5, D4 · vcb → D5, D6 | 30° + 60°·k |

In the three-phase bridge, the top devices are D1 (a), D3 (b), D5 (c) and the bottom devices are D4 (a), D6 (b), D2 (c). The line voltages have a peak of √3·Vm.

For a diode circuit the output is the highest available path voltage. For a thyristor circuit, each path is delayed by α from its natural firing point.

### 4.3 Average output voltage (Theory Vdc)
Let Vm be the **phase peak** voltage and α the firing angle (α = 0 for diodes).

**Continuous conduction (CCM), no freewheeling diode** (valid for any E, because vo is set only by the switching):

| Circuit | Vdc |
|---|---|
| 1φ full-wave bridge | 2·Vm·cos α / π |
| 3φ half-wave | 3√3·Vm·cos α / (2π) |
| 3φ full-wave bridge | 3√3·Vm·cos α / π  (= 3·V_LL,peak·cos α / π) |

The 1φ half-wave circuit has no continuous mode without a freewheeling diode, so no CCM formula applies.

**Output clamped at zero** (resistive load, or a freewheeling diode, with E = 0):

| Circuit | Vdc |
|---|---|
| 1φ half-wave | Vm(1 + cos α) / (2π) |
| 1φ full-wave bridge | Vm(1 + cos α) / π |
| 3φ half-wave | α ≤ 30°: 3√3·Vm·cos α / (2π);  30° < α < 150°: 3·Vm[1 + cos(α + 30°)] / (2π);  α ≥ 150°: 0 |
| 3φ full-wave bridge | α ≤ 60°: 3√3·Vm·cos α / π;  60° < α < 120°: 3√3·Vm[1 + cos(α + 60°)] / π;  α ≥ 120°: 0 |

**Discontinuous, single-phase, inductive load, no freewheeling diode, E = 0:** with β the extinction angle measured from the simulation,

- half-wave: Vdc = Vm(cos α − cos β) / (2π)
- full-wave bridge: Vdc = Vm(cos α − cos β) / π

**Which formula is shown:** CCM or boundary mode uses the CCM formula (or the clamped formula when DF is on). In discontinuous mode it uses the clamped formula when E = 0 and (L = 0 or DF on), the β formula for the single-phase inductive case, and otherwise shows **n/a** because no simple closed form exists.

### 4.4 Measured quantities
Computed over the last full cycle of the simulation:

- **Vdc** = mean(vo), **Idc** = mean(io)
- **Vrms** = √mean(vo²), **Irms** = √mean(io²)
- **Ripple factor** = √(Vrms² − Vdc²) / Vdc
- **Input power factor** (phase a) = P / (Vs,rms · Is,rms), with P = mean(va · ia), Vs,rms = Vm/√2 and Is,rms = rms of the phase-a source current. This is the total power factor, including distortion.

### 4.5 Conduction mode
Classified from the load current over one cycle:

- **DCM (discontinuous):** io = 0 for more than 1 % of the cycle.
- **Boundary (critical):** io never stays at zero but its minimum falls below 3 % of Idc.
- **CCM (continuous):** otherwise.

Useful check: for a single-phase full-wave bridge with E = 0, the boundary occurs at α = φ, where φ = arctan(ωL/R). That is, **L_critical = R·tan α / ω**.

### 4.6 Checks against theory
- Diode circuits with a resistive load match textbook values: single-phase half-wave gives Vdc = Vm/π, Vrms = Vm/2, ripple factor 1.21, PF 0.707; the full-wave bridge gives Vdc = 2Vm/π, ripple factor 0.483, PF 1.
- Across a sweep of 272 combinations (all four circuits, diode and thyristor, several α, L, E and freewheeling diode settings) where a theory value is shown, the simulated Vdc agreed with it to within 1.5 %.

---

## 5. Notes and limitations
- Ideal components: no forward drop, no source inductance (so no commutation overlap), no snubbers, no device turn-off time.
- A half-wave circuit **cannot** reach continuous conduction without a freewheeling diode. This is real behaviour, not a bug.
- If E is larger than the source can supply, no device conducts and io = 0.
- Very large L/R ratios need many cycles to settle. The program starts from an estimate to reduce this, but extreme values (for example 100 H with 1 mΩ) may not be fully settled.
- The gate pulse is fixed at 20°. At large α, a device that is not forward-biased during its pulse will not fire.

---

## 6. Run and deploy

**Locally:** open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari).

**GitHub Pages:**
1. Put `index.html` and `README.md` in a repository.
2. Go to **Settings → Pages**, choose **Deploy from branch**, select `main` and `/ (root)`.
3. The app is served at `https://<username>.github.io/<repository>/`.

---

## 7. Files

| File | Purpose |
|---|---|
| `index.html` | The complete app (HTML, CSS and JavaScript) |
| `README.md` | This document |
