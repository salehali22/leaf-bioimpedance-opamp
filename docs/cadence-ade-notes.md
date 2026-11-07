<sub>[← 4. Full amplifier](04-full-amplifier.md) · [Overview](../README.md) · **Cadence notes**</sub>

# Cadence ADE Explorer notes

Practical things learned while running this project in Virtuoso IC6.1.8 / ADE Explorer / Spectre 23.1 with gpdk045. Useful if you are doing a similar analog design for the first time.

## Setup

| Topic | Note |
|---|---|
| Devices | `pmos1v_3` (model g45p1svt) and `nmos1v_3` (g45n1svt), 3-terminal symbols. Bulk is set by property: PMOS bulk = `vdd!`, NMOS bulk = `gnd!`. |
| Supply net | Must be named `vdd!` (lowercase) to match the bulk property. |
| Width | Set **Fingers** and **Finger Width**. Use one unit finger (2 µm here) everywhere so mirror ratios are exact. The schematic label shows only the total width: press `q` to check fingers. |
| Units | Width and length fields need units (`2u`, `1u`, `100n`). Fingers is a plain number. Frequency fields take `1`, `1G`, not `1Hz`. |
| Diode connection | Gate wired to the device's **own drain**, not to `vdd!`. |

## Sweeps and analyses

| Topic | Note |
|---|---|
| DC sweep resolution | Set Sweep Type **Linear** and pick **Step Size**, otherwise points can be far apart and curves look like straight segments. |
| Several values of a variable | Type several values (`1p 3p 10p`) in the design variable's value field: each one gives a curve. |
| Sweeps override variables | A sweep defined under Parameters overrides the design-variable value. Remove it before single-point runs. |
| AC input | Set **AC magnitude = 1** on the input source, otherwise the AC result is all zero. |
| Open-loop AC trick | `Vout` → 1 GΩ → inverting input, 1 F from that input to ground. DC: closed loop (centred bias). AC: open loop. |
| CMRR trick | Same testbench, move the 1 F capacitor's bottom to the non-inverting input node. |
| Noise | Input voltage source = `/V2` (type the source name, not a wire). Direct Plot → noise → Input Noise, V/√Hz. |
| Temperature error | "Unknown parameter h in temp": a stray character landed in Setup → Temperature. Set it back to 27. |

## Expressions

| Purpose | Expression |
|---|---|
| Small-signal gain from a DC sweep | `deriv(VS("/out"))` (differentiates against whatever is swept) |
| Gain in dB | `dB20(deriv(VS("/out")))` |
| Output resistance at a point | `value((1 / abs(deriv(i("/<SRC>/PLUS" ?result "dc")))) <Vd>)` |
| Gain at one frequency | `value(dB20(VF("/Vout2")) 100000)` |
| Phase at one frequency | `value(phase(VF("/Vout2")) 100000)` |
| Integrated noise | `rmsNoise(100 100000)` |

- Expressions need **Plot** ticked to be evaluated.
- `VAR("Vcm")` returns the fixed design value, not the swept one. Use a node voltage instead (for example `VS("/Vin")`).
- A source's current into its + pin is negative when current leaves that pin.

## Operating point and plots

| Topic | Note |
|---|---|
| Operating point | Tick "Save DC Operating Point". Results → Print → DC Operating Points, then click the device. |
| Annotate schematic | Results → Annotate (DC Node Voltages + DC Operating Points). Clear with View → Annotations → Design Defaults. Press `f` to fit before saving an image. |
| Spectre region codes | 0 off, 1 triode, 2 saturation, 3 subthreshold. |
| PMOS sign | A negative $V_{GS} - V_{TH}$ on a PMOS means $\lvert V_{GS} \rvert > \lvert V_{TH} \rvert$: above threshold. |
| Markers | `v` vertical, `h` horizontal, double-click to type an exact value. Check M (mega) vs m (milli) vs k when typing frequencies. |
| Legends | With many curves the ViVA legend collapses to "more ...": name the curves in the caption or use marker dots. |
| Saving figures | ViVA: File → Save Image, white background. |

## Moving from a prototype model to the real kit

- The Qucs-S prototype used PTM 45 nm models at L = 100 nm. gpdk045 shows much stronger short-channel effects, so L had to go to 1 µm.
- Sizes do not transfer between model sets: the prototype's nulling-resistor device gave only 484 Ω in gpdk045 instead of tens of kΩ.
- Near weak inversion, hand-calculated overdrive is not a valid saturation check. Use the simulator's $V_{DSAT}$ and, better, watch the gain.

---

<sub>[← 4. Full amplifier](04-full-amplifier.md) · [Overview](../README.md) · **Cadence notes**</sub>
