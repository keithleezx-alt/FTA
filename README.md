# Fault Tree Workbench

Two single-page tools for fault tree analysis:

- **Fault Tree Workbench** (`Fault Tree Workbench.html`) turns an Internal Block Diagram (IBD) into a fault tree. You draw the architecture as blocks and one-way connections, give each block failure data, mark redundancy and common-cause groups, group blocks in labelled frames, and connect the blocks whose loss causes the top event. The page then generates the fault tree, the minimal cut sets, the top-event probability and component importance, and updates them all as you edit.
- **Fault Tree Editor** (`Fault Tree Editor.html`) opens a fault tree saved as Open-PSA XML, from the workbench or another tool, and lets you edit the tree itself: probabilities, events, gates and grouping. See [Fault Tree Editor](#fault-tree-editor).

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
8. [Fault Tree Editor](#fault-tree-editor)
9. [Limitations](#limitations)
10. [References](#references)

---

## Quick start

1. Open the workbench. It loads an example hexacopter (battery → PDB → dual IMU, dual GNSS → flight controller → 6-in-1 ESC → six motors → the top event *Loss of thrust*, which occurs when 2 of the 6 motors are lost) with **fake** failure rates. The IMUs and the GNSS receivers sit in labelled frames.
2. Click a block to edit its name, failure model and value in the right-hand panel.
3. Click **Connect**, tap a source block, then tap the block that receives the flow. Connections can run in any direction.
4. Click a connection to set its flow label and, for backup inputs, a **redundancy group**.
5. Connect the blocks whose loss causes the top event to the **TOP EVENT** node, name the undesired event and set the **mission time**.
6. Read the results below the diagram: top-event probability, single-point failures, the fault tree, minimal cut sets and importance ranking.

To start from scratch, click **Clear**, then **Confirm clear**.

### Turning common-cause failures on or off

Common-cause failures (CCF) are optional. Use the **Model common-cause failures** checkbox in the Top event panel:

- **Off:** every block fails independently. The common-cause group field, the alpha-factor panel, the CCF tags and the CCF notes are hidden. This is the simpler model to start with.
- **On:** blocks with the same common-cause group can fail together, using the alpha-factor model (see [Common-cause failures](#common-cause-failures-alpha-factor-model)).

Turning it off doesn't delete anything. Groups and alpha values are kept, ignored in the results, and used again when you turn it back on. The panel says how many blocks still have a group. The setting is saved with the project and can be undone like any other change.

New projects without common-cause groups start with it off. Projects that already have groups, including the example, start with it on, so their results don't change.

---

## Concepts

The workbench needs four inputs to produce a fault tree. An IBD alone isn't enough.

| Input | Where you set it | What it means |
|---|---|---|
| **Architecture** | IBD canvas | Blocks and the one-way flows between them. An arrow from A to B means B needs A's output, whichever way it points on the canvas. |
| **Failure data** | Block inspector | Each block's chance of failing on its own, as a constant rate λ or a fixed probability. |
| **Failure logic** | Connection inspector | Inputs are *required* by default. Inputs into the same block that share a redundancy group back each other up, and the group is lost when at least k of its n inputs are lost (k-of-n). |
| **Top event** | TOP EVENT node | The system-level failure the tree is built for. It's a node of its own, not a component: the blocks connected to it define when it occurs. |

**Frames** add context: a labelled frame around blocks becomes an intermediate event in the tree, such as *Failure of primary GNSS*, without changing any probability.

**Common-cause failures (CCF)** are added on top: blocks that share a common-cause group (e.g. two identical IMUs) can fail together from a shared cause, using the alpha-factor model.

---

## Using the workbench

### File bar

| Button | Action |
|---|---|
| **Load file…** | Open a project `.json`, or a SysML file previously exported from this workbench. |
| **Save .json** | Save the whole project (layout, data, groups, top event). |
| **Undo** / **Redo** | Step back or forward through your changes: blocks, connections, frames, settings, Clear and Reset example included. **Ctrl+Z** undoes; **Ctrl+Shift+Z** or **Ctrl+Y** redoes (**⌘** on a Mac). A burst of typing in one field, or one drag, is a single step. The last 100 steps are kept until you reload the page. While you're typing in a field, the shortcuts undo text in that field instead. |
| **IBD → PowerPoint** | Export the diagram as a widescreen `.pptx` slide built from native, editable shapes: blocks (stereotype, name and failure data), the top-event node, frames, curved connections with arrowheads, ports, flow labels, and the CCF and shared-event (`≡ TAG`) tags. Redundant connections and frames are dashed. |
| **IBD → SVG** | Export the diagram as a vector image. |
| **IBD → SysML v2** | Export the diagram as SysML v2 textual notation. |
| **Fault tree → PowerPoint** | Export the generated tree as a `.pptx` slide built from native, editable shapes (boxes, gates, events, wires and labels), with the top-event probability in the title. |
| **Fault tree → SVG** | Export the generated tree as a vector image. |
| **Fault tree → Open-PSA XML** | Export the tree in the Open-PSA Model Exchange Format, for the [Fault Tree Editor](#fault-tree-editor) and other FTA tools. |

### Diagram toolbar

| Button | Action |
|---|---|
| **+ Block** | Add a block. |
| **+ Frame** | Add a labelled frame in the top-left corner, then drag it over the blocks it should group. The button only works when no block is selected, so a new frame is never tied to one component; click empty canvas to deselect first. See [Frames](#frames). |
| **Connect** | Draw a connection: tap the source, then the target. The top-event node can only be a target. |
| **Tidy** | Line everything up. See [Tidy](#tidy). Ctrl+Z undoes it. |
| **Copy** / **Paste** | Copy the selected block, or a selected frame together with the blocks inside it and the connections between them, then paste a copy offset from the original. **Ctrl+C** / **Ctrl+V** (**⌘C** / **⌘V** on a Mac) work too. Pasting again places further copies. |
| **Delete** | Delete the selected block (and its connections), connection or frame. Deleting a frame keeps its blocks. The top-event node can't be deleted. The **Delete** or **Backspace** key does the same. **Ctrl+X** (**⌘X** on a Mac) cuts instead: it copies the selection, then deletes it, and the next **Ctrl+V** puts it back in place. Cutting a frame removes its blocks too. |
| **Clear** | Remove every block, connection and frame, leaving only the top-event node, for a blank slate. Click **Confirm clear** within 4 seconds to go ahead. Save first if you want to keep your work. |
| **Reset example** | Replace the current project with the example. Save first if you want to keep your work. |

Drag blocks or frames to move them; moving a frame moves the blocks inside it. Drag a frame's bottom-right corner to resize it. Click empty canvas to clear the selection.

**Copies are new, independent components.** A pasted block keeps the original's name and failure data, so rename it. The exception is a block with a shared failure event tag: its copy keeps the tag, so it's the same failure drawn again. That's a quick way to draw a shared component in a second place.

**Connections run in any direction.** Each connection leaves from the side of the source block that faces the target and enters the target on the side facing the source: left or right when the blocks are side by side, top or bottom when one is stacked above the other. Several connections on one side are spread out so they don't overlap. The arrowhead shows the direction of flow.

### Tidy

**Tidy** rearranges the diagram so it's neat and aligned, without changing the analysis:

- **Columns follow the flow.** Blocks are placed left to right by how many inputs lead up to them, and the top event goes in the last column on the right. A block with no inputs, such as a battery or antenna, sits just before the first block that uses it.
- **Rows keep your order.** Within each column, blocks stay in their current top-to-bottom order and are snapped into straight, evenly spaced rows.
- **Frames move as a unit.** Each frame is refitted tightly around its own blocks, and no other block ends up inside it, so every frame keeps the same members. Nested frames stay nested.
- **The canvas can grow.** If the tidied diagram doesn't fit the standard 1300 × 520 canvas, the canvas is enlarged.

The fault tree, probabilities and cut sets are identical before and after.

### Block settings

| Field | Notes |
|---|---|
| **Name** | Shown on the diagram and in the fault tree. |
| **Stereotype** | Free text shown as «stereotype», e.g. `sensor`, `power`. |
| **Failure model** | *Constant failure rate λ* (per hour) or *Fixed probability* (per mission). Switching converts the value so the probability stays the same. |
| **Value** | λ ≥ 0, or a probability from 0 to 1. The line below shows the resulting mission probability. |
| **Common-cause group** | Blocks with the same name form a CCF group. Leave empty for none. Shown only when **Model common-cause failures** is on. |
| **Shared failure event** | Blocks with the same tag are **one physical failure** drawn in several places. They share a single basic event that is counted once. Setting a tag that's already in use copies that block's failure data, and later edits stay in sync. The tag is shown as `≡ TAG` above the block. Leave empty for a block that is its own component. See [Modelling shared components](#modelling-shared-components). |
| **Never fails (P = 0)** | Sets a fixed probability of 0. The block stays in the diagram and tree (dimmed) but never causes a failure itself. |

### Connection settings

| Field | Notes |
|---|---|
| **Flow** | What passes through, e.g. `28V`, `attitude`. Documentation only; it doesn't change the calculation. |
| **Redundancy group** | Empty = required input. Connections into the **same target** with the **same group name** form a redundancy group. Redundant connections are drawn dashed. |
| **Group loss threshold (k)** | Appears when the group has 2 or more inputs. The group is lost when **at least k** of its n inputs are lost. k = n (default) is AND, k = 1 is OR, anything between is a voting gate. The diagram label shows `k/n` when k < n. The setting is shared by every connection in the group. |
| **Reverse direction** | Swaps source and target. Connections into the top event can't be reversed. |

**Example: 2 of 6 rotors.** Connect all six motors to the top-event node, give each connection the redundancy group `Rotors`, then select any one of those connections and set the threshold to 2.

### Top event

The top event is the rounded **TOP EVENT** node on the diagram. It isn't a component and has no failure data, so it never appears as a basic event; it's only the box at the top of the tree.

- **Connect the blocks whose loss causes it.** Required connections mean any one of those losses causes the top event (OR). A redundancy group with threshold k means k of them must be lost.
- **Rename it** in the Top event panel, or select the node.
- **One cause, no extra gate.** If the top event has a single cause, the tree puts the top box directly above that cause's gate, without an extra one-input gate.

| Field | Notes |
|---|---|
| **Undesired event** | The label at the top of the fault tree and on the node. |
| **Mission time (hours)** | Used to convert failure rates to probabilities. |
| **Model common-cause failures** | Shows the common-cause controls and includes the groups in the results. See [Turning common-cause failures on or off](#turning-common-cause-failures-on-or-off). |

Projects saved before the top-event node existed picked a block as the top event. When you load one, a top-event node is added next to that block and connected to it, so the results are unchanged. The block's own failure stays in the tree as a basic event.

### Frames

A frame is a labelled box drawn around blocks, such as a subsystem or a redundant channel. Its label becomes an intermediate event in the fault tree, so the tree reads *Failure of primary GNSS* instead of collecting every low-level failure under one OR gate.

- **Membership:** a block is inside a frame when its centre is.
- **Effect on the tree:** where a block outside the frame depends on blocks inside it, the failures coming out of the frame are gathered under a gate labelled with the frame's text, drawn as a blue box.
- **Gate type:** inside a required input or an AND group, the frame gate has the same type as the gate it sits in, so the logic is unchanged. Inside a k-of-n group, each input from the frame is labelled separately, because a vote can't be split.
- **Nesting:** frames can be nested; the outer frame is applied first, then the inner one.
- **Probability:** frames only regroup the tree. The top-event probability and cut sets are the same with or without them.

Select a frame to edit its label. The inspector lists the blocks inside it.

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
| **Generated fault tree** | Boxes grow taller to fit long descriptions, and every box in a row matches the tallest one so the gates stay aligned. The top event at the top, then OR/AND gates, voting gates (OR shape marked `k/n`), basic events (circles) and transfer symbols (△) where a subtree is reused. A branch that's needed in several places is drawn in full once and tagged **T1**, **T2**…; every other place shows a △ with the same label instead of repeating it. It's one event, counted once. Every gate shows its exact probability. Blue boxes are frame labels. Repeated basic events are drawn with a heavier circle and an `R×n` tag. Red = single-point failure; dashed circle = common-cause event; dimmed = probability 0. |
| **Minimal cut sets table** | Each set with its order (SPF, CCF, ×2, ×3…), probability and share of the total. |
| **Component importance** | Fussell–Vesely: the share of P(top) that disappears if the event can't occur. |

The tree preview starts at **Fit**: it scales to the panel width and the window height, so the whole tree is visible without scrolling. Hover over a box to read its full label and probability.

To read or work on part of a large tree, zoom in:

| Control | Action |
|---|---|
| **+** / **−** | Zoom in or out around the centre of the panel. The percentage shows the current size. |
| **Ctrl + mouse wheel** (**⌘** on a Mac), or a trackpad pinch | Zoom around the pointer. A plain wheel still scrolls the page. |
| Drag | Pan when zoomed in. |
| **Fit** | Return to the whole-tree view. |

The zoom level stays put while you edit. The PowerPoint and SVG exports are always full size, whatever the zoom.

---

## How the analysis works

### Block logic

For every block, its output is lost if **any** of these happen (OR):

- the block fails internally;
- any **required** input is lost;
- **at least k of the n** inputs in a redundancy group are lost (k = n by default, which is AND within the group).

The top event occurs when its inputs are lost, by the same rules, but it has no internal failure of its own.

The fault tree is built by walking backwards from the top-event node through its inputs. A block reached a second time is drawn as a transfer (△) rather than repeated; the full branch and its transfers share a label such as T1. Blocks with no inputs become basic events. Frames then regroup branches under their labels, as described in [Frames](#frames).

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

A basic event is *repeated* when it reaches the top event through more than one path, for example a block that feeds several others, a CCF event shared by several members, or blocks tagged with the same **shared failure event**. Multiplying and adding gate probabilities naively would count such an event more than once.

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

**Modelling this in the workbench.** Draw each component as one block. If a component feeds several branches, such as B and G here, connect that single block to every place that needs it, or draw it more than once and give every copy the same shared failure event tag. See [Modelling shared components](#modelling-shared-components).

### Modelling shared components

Each block is one basic event, so a component that several others depend on is counted once. There are two ways to draw such a component:

- **Draw it once and connect it everywhere it's needed.** If a single power supply, antenna or bus feeds B and C, draw one block and connect it to both. No other setting is needed.
- **Draw it in several places and tag the copies.** Sometimes one block can't reasonably connect to everything, for example a component used by distant parts of the diagram, or one that would make the diagram unreadable. Draw a block in each place and give every copy the same **shared failure event** tag, such as `ANT-1`. The copies become one basic event, their failure data stays in sync, and each shows `≡ ANT-1` above it.

Either way, the fault tree shows the component under every branch it affects, marked `R×n`, and every probability counts it once.

Keep these cases apart:

- **Untagged blocks are always separate components,** even if they have the same name, such as two identical antennas. Their failures are combined as independent events. Copying and pasting an untagged block therefore models a second, independent part and double counts a component that is really shared.
- **Identical parts that can fail together** from a shared cause, such as the same batch, environment or software, are separate blocks in a common-cause group (see [Common-cause failures](#common-cause-failures-alpha-factor-model)), not a shared failure event.

### Loops

Static fault trees can't represent circular dependencies. If the connections form a loop, the analysis cuts it when it revisits a block, so that path contributes nothing and the risk may be **understated**. Keep connections one-way and acyclic. For a shared bus, model the bus as its own block that both ends depend on.

---

## Import and export

### Saving files

When the page is opened directly in a browser, exports download straight away with their normal names (`ibd.sysml`, `fault-tree.xml`).

When the page runs inside Claude, each export asks you to confirm the save. If saving isn't available in that view, the content opens in a panel with a **Copy** button so you can paste it into a file yourself. Two formats are saved there with an extra `.txt` ending, because only certain file types can be saved; rename them after saving:

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

The Open-PSA Model Exchange Format, accepted by SCRAM and other Open-PSA-compatible tools, and by the [Fault Tree Editor](#fault-tree-editor).

- **Same tree as on screen.** The file contains the generated tree: one gate for each gate on the page, including the top event, block gates (`G_<block>_output_lost`), redundancy-group gates and frame gates, each with its label.
- **Basic events:** one per failure event, including CCF events.
- **k-of-n groups** are written as `<atleast min="k">`.
- **Workbench-only details** go in `<attributes>`: `ftw-kind` (top, block, group or frame) on gates, and `ftw-ccf` on common-cause events. Other tools ignore them.
- **Probabilities** are written as **per-mission values**, already converted from rates and already split by the alpha-factor model.

The file names the top gate in a comment at the start. Use the file to edit the tree in the editor, or to check the results with an independent solver.

### Browser storage

The current project is kept in your browser automatically (key `ft-workbench-v2`). It stays on that browser only: it doesn't sync across devices and is lost if site data is cleared. Save a `.json` for anything you want to keep.

---

## Project file format

```json
{
  "top": "thrust",
  "topText": "Loss of thrust",
  "T": 1,
  "ccfOn": true,
  "ccf": { "IMU": [0.95, 0.05] },
  "vote": { "thrust|Rotors": 2 },
  "blocks": [
    { "id": "imua", "name": "IMU A", "st": "sensor",
      "mode": "rate", "v": 5e-5, "ccf": "IMU", "x": 360, "y": 40 },
    { "id": "thrust", "kind": "top", "name": "Loss of thrust", "x": 1150, "y": 220 }
  ],
  "links": [
    { "id": "l5", "from": "imua", "to": "fc", "flow": "attitude", "group": "IMU" }
  ],
  "frames": [
    { "id": "fimu", "label": "Loss of attitude data", "x": 344, "y": 16, "w": 160, "h": 184 }
  ]
}
```

| Field | Type | Meaning |
|---|---|---|
| `top` | string | Id of the top-event node. |
| `topText` | string | Top event label (the same as the top-event node's name). |
| `T` | number > 0 | Mission time in hours. |
| `ccfOn` | boolean | Whether common-cause groups are modelled. Missing = on if any block has a group. |
| `ccf` | object | CCF group name → array of alpha values α₁ … αₘ. |
| `vote` | object | `"<target block id>|<redundancy group>"` → k, the number of lost inputs that loses the group. Missing = all (n). |
| `blocks[].id` | string | Unique id. |
| `blocks[].kind` | `"top"` or empty | `"top"` marks the top-event node. Exactly one block has it; its failure fields are ignored. |
| `blocks[].name` / `st` | string | Name and stereotype. |
| `blocks[].mode` | `"rate"` or `"prob"` | Failure model. |
| `blocks[].v` | number | λ per hour (`rate`) or probability 0–1 (`prob`). |
| `blocks[].ccf` | string | Common-cause group name, or empty. |
| `blocks[].rep` | string | Shared failure event tag, or empty. Blocks with the same tag share one basic event. |
| `blocks[].x`, `y` | number | Position on the canvas. |
| `cw`, `ch` | number | Canvas size, when Tidy has enlarged it beyond 1300 × 520. |
| `links[].from` / `to` | string | Source and target block ids. |
| `links[].flow` | string | Flow label. |
| `links[].group` | string | Redundancy group name, or empty for a required input. |
| `frames[].label` | string | Text of the frame, used as the intermediate event in the tree. |
| `frames[].x`, `y`, `w`, `h` | number | Position and size of the frame on the canvas. |

Files from the first version (with `p` instead of `mode`/`v`) load as constant rates. Files without a top-event node get one, connected to their old top block.

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

## Fault Tree Editor

`Fault Tree Editor.html` edits a fault tree directly, without the block diagram behind it. Use it to adjust a generated tree for a report, try what-if probabilities, or rework a tree from another Open-PSA tool. It recalculates as you edit, using the same exact BDD method as the workbench.

### Opening a tree

1. In the workbench, click **Fault tree → Open-PSA XML**.
2. In the editor, click **Open XML…** and choose the file, or drop it onto the page.

The editor opens the top gate named in the workbench's comment, or else the gate that no other gate uses. It also reads files from other Open-PSA tools if they use OR, AND and at-least gates (nested formulas included), and basic events given as a `float`, a `parameter` or an `exponential` of two floats. Anything else is reported when the file opens. **New tree** starts a tree with one gate and one basic event.

The tree is kept in your browser automatically (key `ft-editor-v1`). Changes don't flow back to the workbench project: the editor works on the tree, not the IBD.

### Editing

Tap any box in the tree to select it. The panel on the right shows what you can change. The tree has the same zoom controls as the workbench (**−**, **Fit**, **+**, Ctrl + wheel, drag to pan). **Selection**, or a double-click on a box, zooms to 100% centred on that box.

| Selected | What you can do |
|---|---|
| **Top event or gate** | Edit the description. Change the type: OR, AND, or at least k of the inputs. Reorder (↑ ↓) or remove (✕) inputs. Add a new basic event or gate, or an existing one. Tick several inputs and click **Group checked under new gate** to put them under a new labelled intermediate event. |
| **Basic event** | Edit the description and the probability per mission. |
| **Anything but the top event** | **Insert gate above** wraps it in a new gate. **Remove from parent** takes it out of the gate it was selected under. |

The **Basic events** table lists every event with editable description and probability fields, sorted by importance, so you can change several probabilities quickly.

- **Undo and Redo** (**Ctrl+Z**, **Ctrl+Shift+Z** or **Ctrl+Y**) cover every change.
- **Unused items are removed.** An event or gate that's no longer used anywhere is deleted automatically.
- **Loops are blocked.** The "add existing" list never offers a gate that would create a loop.

**Adding labels without changing the result.** Grouping inputs of an OR gate under a new OR gate, or inputs of an AND gate under a new AND gate, adds an intermediate event but leaves every probability unchanged. Grouping is not offered for several inputs of an at-least gate, because that would change the vote. Changing a gate's type, or adding and removing inputs, does change the logic.

**Repeated events.** Adding an existing event under a second gate makes it a repeated event: it's drawn in both places, marked `R×n`, and counted once. A gate used in more than one place is drawn in full once, tagged **T1**, **T2**…, and shown elsewhere as a transfer (△) with the same label.

### Results and export

- **Results:** the top-event probability, single-point failures, repeated events and minimal cut sets, as in the workbench. Every gate shows its exact probability. An empty gate is outlined in red and never occurs until it has inputs.
- **Exports:** Open-PSA XML (which the editor and the workbench's other tools can read), PowerPoint (editable shapes, as in the workbench) and SVG.

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
- **Shared failure events** use the first tagged block's common-cause group; groups set on the other tagged blocks are ignored.
- **PowerPoint export** loads the PptxGenJS library from cdn.jsdelivr.net the first time it's used, so it needs an internet connection. The IBD is scaled to fit one widescreen slide. Very large fault trees are scaled down to PowerPoint's 56-inch slide limit.
- **SysML import** only reads files exported by this workbench.
- **Frames use block centres**, so a block only half inside a frame may or may not belong to it. Frames don't regroup the inputs of a k-of-n group into one sub-gate.
- **The editor and the workbench are separate.** Edits made in the Fault Tree Editor don't update the workbench project, and the editor supports OR, AND and at-least gates only (no NOT, XOR or house events).

---

## References

- Quanterion Solutions, *Nonelectronic Parts Reliability Data (NPRD-2023)*.
- Quanterion Solutions, *Failure Mode/Mechanism Distributions (FMD-2016)*.
- US NRC, *Common-Cause Failure Parameter Estimations* (NUREG/CR-5497 and updates).
- IEC 61508-6:2010, Annex D, *Methodology for quantifying the effect of hardware-related common cause failures*.
- SAE ARP4761A, *Guidelines for Conducting the Safety Assessment Process on Civil Aircraft, Systems, and Equipment*.
- Open-PSA Model Exchange Format, https://open-psa.github.io
- OMG Systems Modeling Language (SysML) v2 specification.
