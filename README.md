# Fault Tree Workbench

A single-page tool that turns an Internal Block Diagram (IBD) into a fault tree. You draw the architecture as blocks and one-way connections, give each block failure data, mark redundancy and common-cause groups, and pick a top event. The page then generates the fault tree, the minimal cut sets, the top-event probability and component importance, and updates them all as you edit.

It's meant for early design trade studies and for teaching model-based safety analysis. It is **not** a certified safety tool (see [Limitations](#limitations)).

---

## Contents

1. [Quick start](#quick-start)
2. [Concepts](#concepts)
3. [Using the workbench](#using-the-workbench)
4. [How the analysis works](#how-the-analysis-works)
5. [Import and export](#import-and-export)
6. [Project file format](#project-file-format)
7. [Getting failure data](#getting-failure-data)
8. [Limitations](#limitations)
9. [References](#references)

---

## Quick start

1. Open the workbench. It loads an example hexacopter (battery → PDB → dual IMU, dual GNSS → flight controller → 6-in-1 ESC → six motors → thrust, which is lost when 2 of the 6 motors are lost) with **fake** failure rates.
2. Click a block to edit its name, failure model and value in the right-hand panel.
3. Click **Connect**, tap a source block, then tap the block that receives the flow.
4. Click a connection to set its flow label and, for backup inputs, a **redundancy group**.
5. Choose the **top event** block and set the **mission time**.
6. Read the results below the diagram: top-event probability, single-point failures, the fault tree, minimal cut sets and importance ranking.

Your work is saved automatically in your browser. Use **Save .json** to keep a copy you can move between machines or share.

---

## Concepts

The workbench needs four inputs to produce a fault tree. An IBD alone isn't enough.

| Input | Where you set it | What it means |
|---|---|---|
| **Architecture** | IBD canvas | Blocks and the one-way flows between them. An arrow from A to B means B needs A's output. |
| **Failure data** | Block inspector | Each block's chance of failing on its own, as a constant rate λ or a fixed probability. |
| **Failure logic** | Connection inspector | Inputs are *required* by default. Inputs into the same block that share a redundancy group back each other up, and the group is lost when at least k of its n inputs are lost (k-of-n). |
| **Top event** | Top event panel | The system-level failure the tree is built for, defined as the loss of one block's output. |

**Common-cause failures (CCF)** are added on top: blocks that share a common-cause group (e.g. two identical IMUs) can fail together from a shared cause, using the alpha-factor model.

---

## Using the workbench

### File bar

| Button | Action |
|---|---|
| **Load file…** | Open a project `.json`, or a SysML file previously exported from this workbench. |
| **Save .json** | Save the whole project (layout, data, groups, top event). |
| **IBD → PowerPoint** | Export the diagram as a widescreen `.pptx` slide built from native, editable shapes: blocks (stereotype, name and failure data), curved connections with arrowheads, ports, flow labels, and the TOP and CCF tags. Redundant connections are dashed. |
| **IBD → SVG** | Export the diagram as a vector image. |
| **IBD → SysML v2** | Export the diagram as SysML v2 textual notation. |
| **Fault tree → PowerPoint** | Export the generated tree as a `.pptx` slide built from native, editable shapes (boxes, gates, events, wires and labels), with the top-event probability in the title. |
| **Fault tree → SVG** | Export the generated tree as a vector image. |
| **Fault tree → Open-PSA XML** | Export the tree in the Open-PSA Model Exchange Format for other FTA tools. |

### Diagram toolbar

| Button | Action |
|---|---|
| **+ Block** | Add a block. |
| **Connect** | Draw a connection: tap the source, then the target. |
| **Delete** | Delete the selected block (and its connections) or the selected connection. |
| **Reset example** | Replace the current project with the example. Save first if you want to keep your work. |

Drag blocks to move them. Click empty canvas to clear the selection.

### Block settings

| Field | Notes |
|---|---|
| **Name** | Shown on the diagram and in the fault tree. |
| **Stereotype** | Free text shown as «stereotype», e.g. `sensor`, `power`. |
| **Failure model** | *Constant failure rate λ* (per hour) or *Fixed probability* (per mission). Switching converts the value so the probability stays the same. |
| **Value** | λ ≥ 0, or a probability from 0 to 1. The line below shows the resulting mission probability. |
| **Common-cause group** | Blocks with the same name form a CCF group. Leave empty for none. |
| **Set as top event** | Makes this block's output loss the top event. |
| **Never fails (P = 0)** | Sets a fixed probability of 0. The block stays in the diagram and tree (dimmed) but never causes a failure itself. |

### Connection settings

| Field | Notes |
|---|---|
| **Flow** | What passes through, e.g. `28V`, `attitude`. Documentation only; it doesn't change the calculation. |
| **Redundancy group** | Empty = required input. Connections into the **same target** with the **same group name** form a redundancy group. Redundant connections are drawn dashed. |
| **Group loss threshold (k)** | Appears when the group has 2 or more inputs. The group is lost when **at least k** of its n inputs are lost. k = n (default) is AND, k = 1 is OR, anything between is a voting gate. The diagram label shows `k/n` when k < n. The setting is shared by every connection in the group. |

**Example: 2 of 6 rotors.** Add a block such as `Thrust` (a fixed probability of 0 works well, since it's a function rather than hardware). Connect all six motors to it, give each connection the redundancy group `Rotors`, then select any one of those connections and set the threshold to 2.

### Top event panel

| Field | Notes |
|---|---|
| **Undesired event** | The label at the top of the fault tree. |
| **Block** | Whose output loss defines the top event. |
| **Mission time (hours)** | Used to convert failure rates to probabilities. |

### Common-cause groups panel

Lists each CCF group with editable **α₁ … αₘ** values (m = number of members, 2 to 6) and the resulting Qₜ, Q₁ … Qₘ. If the α values don't add up to 1 they are normalised, and a warning is shown.

### Results

| Section | What it shows |
|---|---|
| **Top-event probability** | P per mission, labelled *exact* or *rare-event approx.* |
| **Repeated events** | Basic events that appear under more than one branch of the tree. Each is counted once. |
| **Occurs about once in** | 1 / P missions. |
| **Minimal cut sets** | Count of the smallest event combinations that cause the top event. |
| **Single-point failures** | Events that cause the top event on their own. |
| **Generated fault tree** | OR/AND gates, voting gates (OR shape marked `k/n`), basic events (circles) and transfer symbols (△) where a subtree is reused. Every gate shows its exact probability. Repeated basic events are drawn with a heavier circle and an `R×n` tag. Red = single-point failure; dashed circle = common-cause event; dimmed = probability 0. |
| **Minimal cut sets table** | Each set with its order (SPF, CCF, ×2, ×3…), probability and share of the total. |
| **Component importance** | Fussell–Vesely: the share of P(top) that disappears if the event can't occur. |

The tree preview always scales to fit the panel width and the window height, so the whole tree is visible without scrolling. Hover over a box to read its full label and probability. Use the PowerPoint or SVG export to see a large tree at full size.

---

## How the analysis works

### Block logic

For every block, its output is lost if **any** of these happen (OR):

- the block fails internally;
- any **required** input is lost;
- **at least k of the n** inputs in a redundancy group are lost (k = n by default, which is AND within the group).

The fault tree is built by walking backwards from the top-event block through its inputs. A block reached a second time is drawn as a transfer (△) rather than repeated. Blocks with no inputs become basic events.

### Probabilities

- Constant failure rate: **P = 1 − e^(−λT)**, where T is the mission time.
- Fixed probability: used as is.
- Basic events are assumed independent, apart from the common-cause events described below.

### Common-cause failures (alpha-factor model)

For a group of m members with alpha factors α₁ … αₘ:

- αₜ = Σ k·αₖ
- Qₜ = the average of the members' probabilities
- Qₖ = k / C(m−1, k−1) · αₖ / αₜ · Qₜ, the probability of each **specific** event that fails exactly k members together

Each member's own basic event is replaced by its **independent** part, α₁ / αₜ × P(member), and one CCF basic event is added for every combination of 2 or more members. A member fails if its independent event or any CCF event that includes it occurs.

A group of 3, for example, gets three pair events and one triple event.

### Minimal cut sets

Cut sets are built bottom-up (OR = union of lists, AND = cross-product) and reduced to minimal sets by removing duplicates and supersets. A k-of-n group is expanded as the OR of the AND of every k-member subset, so 2 of 6 produces C(6,2) = 15 combinations. Events with probability 0 are left out. Each AND expansion is capped at 6,000 combinations to keep the page responsive.

In the example, the 15 motor-pair cut sets sum to 6.0 × 10⁻⁹ per hour, which matches the binomial result P(≥2 of 6) = 1 − (1−p)⁶ − 6p(1−p)⁵ with p ≈ 2 × 10⁻⁵.

### Repeated events and top-event probability

A basic event is *repeated* when it reaches the top event through more than one path, for example a block that feeds several others, or a CCF event shared by several members. Multiplying and adding gate probabilities naively would count such an event more than once.

The workbench avoids this by building a **binary decision diagram (BDD)** of the whole logic, where every basic event is one variable. The top-event probability, every gate probability in the tree and the Fussell–Vesely importance are computed exactly from the BDD, so each event is counted once, however many branches it appears in. There is no limit on the number of events.

Example: A feeds both B and C, and the top event needs both B and C lost. With P(A) = 0.1, P(B) = 0.2 and P(C) = 0.3, the exact result is 0.1 + 0.9 × 0.2 × 0.3 = 0.154. Treating the two branches as independent would give 0.193.

If a model is too large for the BDD (more than 300,000 nodes), P falls back to the **rare-event approximation** (sum of cut-set probabilities, capped at 1), which overestimates P, and the readout says so.

### Worked example: comparing with a hand calculation

This example follows a typical hand calculation and shows which steps the workbench performs automatically.

**The equation.** Expanding a fault tree gives this Boolean equation:

```
[(A·B) + (C + B + D + E)] · [(B + F + G) + G] · H
```

B appears in both brackets and G appears twice in the second, so both are repeated events.

**Hand simplification.**

1. First bracket: B + A·B = B (absorption law, X + X·Y = X), which gives B + C + D + E.
2. Second bracket: G + G = G (idempotent law, X + X = X), which gives B + F + G.
3. Combine: (B + C + D + E) · (B + F + G) = B + (C + D + E)·(F + G) (distributive law, (X + Y)·(X + Z) = X + Y·Z).

The result is:

```
[B + (C + D + E)·(F + G)] · H
```

**What the workbench does instead.** It derives the logic from the block diagram rather than from a typed equation, and applies the same laws while building the minimal cut sets:

| Hand step | Workbench equivalent |
|---|---|
| Idempotent law (G + G = G) | Each event appears once in a cut set, so G·G becomes G. |
| Absorption law (B + A·B = B) | Any cut set that contains a smaller cut set is removed, so {A, B} is dropped because {B} exists. |
| Distributive law and final form | The minimal cut sets are the simplified equation written out as a sum of products. |

For this equation the cut-set table lists 7 minimal cut sets: {B, H}, {C, F, H}, {C, G, H}, {D, F, H}, {D, G, H}, {E, F, H} and {E, G, H}. Event A drops out, as it does in step 1.

**Probabilities.** With these per-mission values:

| Event | A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|---|
| P | 2.56E−06 | 1.10E−03 | 1.16E−03 | 5.36E−06 | 1.20E−03 | 1.10E−03 | 1.10E−02 | 7.77E−04 |

| Method | P(top) |
|---|---|
| Hand calculation: + as addition, · as multiplication (rare-event approximation) | 8.7694 × 10⁻⁷ |
| Sum of the 7 minimal cut-set probabilities | 8.7694 × 10⁻⁷ |
| **Workbench (exact, BDD)** | **8.7688 × 10⁻⁷** |
| Brute force over all 256 event combinations of the original equation | 8.7688 × 10⁻⁷ |
| Unsimplified equation with + and · applied directly | 6.5 × 10⁻⁸ (wrong) |

What the comparison shows:

- **The hand result is right to 3 significant figures** (8.77 × 10⁻⁷). Treating + as addition is the rare-event approximation: it ignores the small chance that two cut sets occur together, so it always overestimates slightly. The workbench uses the exact rule P(X or Y) = P(X) + P(Y) − P(X)·P(Y). The difference is negligible here but grows for probabilities around 10⁻² and above.
- **Simplifying first is essential by hand.** Putting numbers into the unsimplified equation multiplies the two B terms as if they were independent failures, which gives a result about 13 times too low. The workbench avoids this automatically because each event is a single variable, however many branches it appears in, so no hand simplification is needed.

**Modelling this in the workbench.** Draw each component as one block. If a component feeds several branches, such as B and G here, connect that single block to every place that needs it. See [Modelling shared components](#modelling-shared-components).

### Modelling shared components

Shared failures come straight from the architecture. Each block is one basic event, so a component that several others depend on is counted once automatically:

- **Shared component: draw it once.** If a single power supply, antenna or bus feeds B and C, draw one block and connect it to both. The tree then shows the component under each branch it affects (marked `R×n`), and every probability counts it once.
- **Separate components: draw them separately.** Two blocks are always two independent components, even if they have the same name, such as two identical antennas. Their failures are combined as independent events.
- **Don't duplicate a block to tidy the diagram.** Drawing the same physical part twice makes it two independent failures, which double counts it. Connect the one block to every place that needs it instead, even if the lines cross.
- **Identical parts that can fail together** from a shared cause, such as the same batch, environment or software, are a different case. Model them as separate blocks in a common-cause group (see [Common-cause failures](#common-cause-failures-alpha-factor-model)).

### Loops

Static fault trees can't represent circular dependencies. If the connections form a loop, the analysis cuts it when it revisits a block, so that path contributes nothing and the risk may be **understated**. Keep connections one-way and acyclic. For a shared bus, model the bus as its own block that both ends depend on.

---

## Import and export

### Saving files

Each export asks you to confirm the save. If saving isn't available in your view, the content opens in a panel with a **Copy** button so you can paste it into a file yourself.

Two formats are saved with an extra `.txt` ending, because only certain file types can be saved from the page. Rename them after saving:

| Saved as | Rename to |
|---|---|
| `ibd.sysml.txt` | `ibd.sysml` |
| `fault-tree.opsa.xml.txt` | `fault-tree.xml` |

### SVG (IBD and fault tree)

Light-themed, self-contained vector images with styles embedded, suitable for reports and slides. The IBD export is cropped to the blocks and omits the canvas grid and selection highlights.

### SysML v2

Textual notation with:

- one `item def` and `port def` per flow type;
- one `part def` per block, with attributes `failureRate` or `fixedFailureProbability`, `commonCauseGroup`, and an out/in (conjugated) port per connection;
- a `lossThreshold_<group> : Integer` attribute on any block whose redundancy group uses k < n;
- a `part system` holding a part usage for every block, a `connection … connect` for every link, `redundancyGroup` attributes on redundant connections, the `missionTime` attribute, and the CCF alpha values as comments.

The file also carries the full project in a comment (`ft-layout:…`), which is how **Load file…** restores it with block positions intact. The workbench can't read SysML written by other tools. The export is a starting point for import into a SysML v2 tool; check it against your tool's library before relying on it.

### Open-PSA XML

The Open-PSA Model Exchange Format, accepted by SCRAM and other Open-PSA-compatible tools. It contains one gate per block (`G_<block>_lost`) and one basic event per failure event, including CCF events. k-of-n groups are written as `<atleast min="k">`. Probabilities are written as **per-mission values**, already converted from rates and already split by the alpha-factor model. Use it to check the results with an independent solver.

### Browser storage

The current project is kept in your browser automatically (key `ft-workbench-v2`). It stays on that browser only: it doesn't sync across devices and is lost if site data is cleared. Save a `.json` for anything you want to keep.

---

## Project file format

```json
{
  "top": "mot",
  "topText": "Loss of thrust control",
  "T": 1,
  "ccf": { "IMU": [0.95, 0.05] },
  "vote": { "thrust|Rotors": 2 },
  "blocks": [
    { "id": "imua", "name": "IMU A", "st": "sensor",
      "mode": "rate", "v": 5e-5, "ccf": "IMU", "x": 350, "y": 40 }
  ],
  "links": [
    { "id": "l5", "from": "imua", "to": "fc", "flow": "attitude", "group": "IMU" }
  ]
}
```

| Field | Type | Meaning |
|---|---|---|
| `top` | string | Block id of the top event. |
| `topText` | string | Top event label. |
| `T` | number > 0 | Mission time in hours. |
| `ccf` | object | CCF group name → array of alpha values α₁ … αₘ. |
| `vote` | object | `"<target block id>|<redundancy group>"` → k, the number of lost inputs that loses the group. Missing = all (n). |
| `blocks[].id` | string | Unique id. |
| `blocks[].name` / `st` | string | Name and stereotype. |
| `blocks[].mode` | `"rate"` or `"prob"` | Failure model. |
| `blocks[].v` | number | λ per hour (`rate`) or probability 0–1 (`prob`). |
| `blocks[].ccf` | string | Common-cause group name, or empty. |
| `blocks[].x`, `y` | number | Position on the 1300 × 520 canvas. |
| `links[].from` / `to` | string | Source and target block ids. |
| `links[].flow` | string | Flow label. |
| `links[].group` | string | Redundancy group name, or empty for a required input. |

Files from the first version (with `p` instead of `mode`/`v`) load as constant rates.

---

## Getting failure data

| Parameter | Typical source |
|---|---|
| Component failure rates λ | **NPRD-2023** (nonelectronic parts), MIL-HDBK-217 or similar for electronics, supplier data, your own field data. |
| Failure mode split (lost vs wrong output) | **FMD-2016**, Quanterion's companion to NPRD. Not modelled in this version. |
| Alpha factors | Your own maintenance and incident records; US NRC common-cause parameter estimates (originally NUREG/CR-5497) by analogous component type; or a beta factor from **IEC 61508-6 Annex D**. |

**From your own records:** count failure events that hit exactly k members of the group, nₖ, then αₖ = nₖ / Σ nᵢ.

**From a beta factor (pairs only):** α₂ = β / (2 − β), α₁ = 1 − α₂. For example, β = 5% gives α₁ ≈ 0.974, α₂ ≈ 0.026. For groups of three or more, there's no unique conversion from a single β.

**Don't confuse NPRD's α and β with CCF factors.** In NPRD they are the Weibull characteristic life and shape parameters, which are unrelated.

---

## Limitations

- **Not a certified tool.** Use it for early design and trade studies; verify safety-case numbers with a qualified tool and process (e.g. ARP4761, ISO 26262, IEC 61508).
- **One failure mode per block.** Every failure is treated as "output lost". Erroneous output, late output and other modes aren't distinguished, and flow types don't change the logic.
- **Static fault trees only.** No sequence dependencies, spares, repair or cross-monitoring; those need dynamic fault trees or Markov models.
- **Loops are cut**, which may understate risk (see [Loops](#loops)).
- **Independence assumed** apart from the CCF groups you define. Shared hidden causes (software, environment, maintenance) must be modelled explicitly.
- **CCF groups** are limited to 2–6 members, and Qₜ uses the members' average probability, which is most accurate when members are identical.
- **Constant failure rates**: wear-out and infant mortality aren't modelled.
- **k-of-n is count-based.** It treats all inputs in the group as equal. Losing two *adjacent* rotors on a hexacopter is often worse than losing two opposite ones; to capture that, model the specific bad pairs explicitly instead.
- **Renaming a redundancy group** resets its threshold to k = n.
- **Performance**: large or highly redundant models may hit the 6,000-combination cap in the cut-set table, or fall back to the rare-event approximation. The exact probability doesn't depend on the cut-set cap.
- **PowerPoint export** loads the PptxGenJS library from cdn.jsdelivr.net the first time it's used, so it needs an internet connection. The IBD is scaled to fit one widescreen slide. Very large fault trees are scaled down to PowerPoint's 56-inch slide limit.
- **SysML import** only reads files exported by this workbench.

---

## References

- Quanterion Solutions, *Nonelectronic Parts Reliability Data (NPRD-2023)*.
- Quanterion Solutions, *Failure Mode/Mechanism Distributions (FMD-2016)*.
- US NRC, *Common-Cause Failure Parameter Estimations* (NUREG/CR-5497 and updates).
- IEC 61508-6:2010, Annex D, *Methodology for quantifying the effect of hardware-related common cause failures*.
- SAE ARP4761A, *Guidelines for Conducting the Safety Assessment Process on Civil Aircraft, Systems, and Equipment*.
- Open-PSA Model Exchange Format, https://open-psa.github.io
- OMG Systems Modeling Language (SysML) v2 specification.
