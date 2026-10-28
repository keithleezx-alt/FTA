# Architecture Workbench

A single-page tool for fault tree analysis from a system architecture. You draw the system as it really is: blocks, flows in one or both directions, and frames for subsystems. Then you define one or more **analyses**, each naming an undesired event and the blocks whose loss causes it. The page generates the fault tree, minimal cut sets, exact probabilities and component importance for the analysis you select, and updates them as you edit. The diagram itself stays neutral.

Open `Architecture Workbench.html` in a browser; nothing needs installing. A companion page, `Fault Tree Editor.html`, opens the fault trees it exports so you can edit a tree directly (see [Fault Tree Editor](#fault-tree-editor)).

It's meant for early design trade studies and for teaching model-based safety analysis. It is **not** a certified safety tool (see [Limitations](#limitations)).

---

## Contents

1. [Quick start](#quick-start)
2. [Concepts](#concepts)
3. [Drawing the architecture](#drawing-the-architecture)
4. [Analyses](#analyses)
5. [Results](#results)
6. [How the analysis works](#how-the-analysis-works)
7. [Modelling guidance](#modelling-guidance)
8. [Worked examples](#worked-examples)
9. [Import and export](#import-and-export)
10. [Project file format](#project-file-format)
11. [Getting failure data](#getting-failure-data)
12. [Fault Tree Editor](#fault-tree-editor)
13. [Limitations](#limitations)
14. [References](#references)

---

## Quick start

1. **Open the page.** It loads an example hexacopter with **fake** failure rates: battery → PDB → dual IMU and dual GNSS → flight controller ↔ 6-in-1 ESC → six motors. The flight controller and ESC exchange PWM and telemetry over a two-way connection, and the sensors sit in labelled frames.
2. **Draw or edit the architecture.** Click a block to set its name and failure data in the editor under the canvas. Click **Connect**, tap a source block, then tap the block that receives the flow.
3. **Set connection logic.** Click a connection to set its flow label, mark backup inputs with a **redundancy group**, or make it **two-way** and say what each end needs.
4. **Define an analysis.** In the **Analysis** panel, name the undesired event, choose **any**, **all** or **at least k**, and add the blocks whose loss causes it. You can add them from the list or with **Pick on diagram**.
5. **Read the results** below the diagram: probability, single-point failures, the fault tree, minimal cut sets and importance.
6. **Ask more questions of the same diagram** with **+ New analysis**. Switch between analyses with **Showing**.

To start from scratch, click **Clear**, then **Confirm clear**.

---

## Concepts

| Input | Where you set it | What it means |
|---|---|---|
| **Architecture** | Canvas | Blocks and the flows between them. A one-way arrow from A to B means B needs A's output, whichever way it points on the canvas. A two-way connection says separately what each end needs. |
| **Failure data** | Block panel | Each block's chance of failing on its own, as a constant rate λ or a fixed probability. |
| **Failure logic** | Connection panel | Inputs are *required* by default. Inputs into the same block that share a **redundancy group** back each other up; the group is lost when at least k of its n inputs are lost (k-of-n). |
| **Analysis** | Analysis panel | The undesired event at the top of the tree, and which blocks must be lost for it to happen. A project can have several. |

Three optional layers add context or detail:

- **Frames** group blocks under a labelled intermediate event in the tree, such as *Loss of attitude data*, and can show a subsystem's combined probability.
- **Shared failure events** make several drawn blocks one physical failure.
- **Common-cause failures (CCF)** let similar blocks fail together from a shared cause, using the alpha-factor model. They're optional and off for new projects.

---

## Drawing the architecture

### File bar

| Group | Button | Action |
|---|---|---|
| Project | **Load file…** | Open a project `.json` or a SysML file exported from this page. |
| | **Save .json** | Save the whole project as `architecture-project.json`. |
| Edit | **Undo** / **Redo** | Step back or forward through every change. See [Keyboard shortcuts](#keyboard-shortcuts). |
| Architecture | **PowerPoint**, **SVG**, **SysML v2** | Export the diagram. See [Import and export](#import-and-export). |
| Fault tree | **PowerPoint**, **SVG**, **Open-PSA XML** | Export the fault tree of the analysis being shown. |

### Diagram toolbar

| Button | Action |
|---|---|
| **+ Block** | Add a block. |
| **+ Frame** | Add a labelled frame in the top-left corner, then drag it over the blocks it should group. It only works with no block selected, so a frame is never tied to one component; click empty canvas to deselect first. |
| **Connect** | Draw a connection: tap the source, then the target. Drawing over an existing connection in the reverse direction makes it two-way. |
| **Tidy** | Line everything up in columns that follow the flow. See [Tidy](#tidy). |
| **Copy** / **Paste** | Copy the selected block, or a selected frame with its blocks and the connections between them, and paste a copy offset from the original. |
| **Delete** | Delete the selected block (and its connections), connection or frame. Deleting a frame keeps its blocks. A deleted block is also removed from every analysis. |
| **Clear** | Remove every block, connection and frame and start with one empty analysis. Click **Confirm clear** within 4 seconds. |
| **Reset example** | Replace the project with the example. |

Save first if you want to keep your work before **Clear** or **Reset example**. Both can also be undone.

**Moving and sizing:**
- Drag blocks or frames to move them. Moving a frame moves the blocks inside it.
- Drag a frame's bottom-right corner to resize it.
- Click empty canvas to clear the selection.

**Copies are new, independent components.** A pasted block keeps the original's name and failure data, so rename it. The exception is a block with a shared failure event tag: its copy keeps the tag, so it's the same failure drawn again.

### Keyboard shortcuts

Use **⌘** instead of **Ctrl** on a Mac. Shortcuts are ignored while you're typing in a field.

| Keys | Action |
|---|---|
| **Ctrl+C** / **Ctrl+V** | Copy / paste. |
| **Ctrl+X** | Cut: copy, then delete. The next paste puts it back in place. Cutting a frame removes its blocks too. |
| **Delete** or **Backspace** | Delete the selection. |
| **Ctrl+Z** | Undo. A burst of typing in one field, or one drag, is a single step. The last 100 steps are kept until you reload the page. |
| **Ctrl+Shift+Z** or **Ctrl+Y** | Redo. |
| **Esc** | Finish picking blocks, or close an export panel. |

### The canvas

- **Blocks fit their text.** Each block is as wide as its name, stereotype and failure data need, up to 240 px. Longer names wrap and the block grows taller.
- **Connections run in any direction.** Each connection leaves the side of the source that faces the target and enters the target on the side facing the source. Several connections on one side are spread out.
- **Connections route around blocks.** A connection is a smooth curve where the way is clear. If the curve would pass under a block, it becomes a right-angled path around the blocks. Flow labels are placed where they don't cover a block, another label or a frame tag. Overlapping blocks can't be routed around; press **Tidy** to separate them.
- **More room.**

  | Control | Action |
  |---|---|
  | **+** at each edge | Adds space on that side. Adding on the left or top moves the diagram over, so nothing is cut off. |
  | Dragging past the right or bottom edge | Grows the canvas as you drag a block or frame, or resize a frame. The view scrolls with you. |
  | **Trim** | Removes empty space, down to the 1300 × 520 minimum. |

  The diagram keeps its size on screen as the canvas grows; the panel scrolls instead.

### Tidy

**Tidy** makes the diagram neat and aligned without changing any result:

- **Columns follow the flow.**
  - Blocks are placed left to right by how many inputs lead up to them. Two-way connections count in the direction they were drawn.
  - A block with no inputs, such as a battery or antenna, sits just before the first block that uses it.
  - Each column is as wide as its widest block.
- **Rows keep your order.** Within a column, blocks keep their top-to-bottom order and snap into straight rows.
- **Frames move as a unit.** Each frame is refitted around its own blocks and no other block ends up inside it, so frames keep their members. Nested frames stay nested.
- **The canvas is resized** to fit the layout.

### Block settings

Selecting a block, connection or frame opens its settings in the editor directly under the canvas, laid out in rows.


| Field | Notes |
|---|---|
| **Name** | Shown on the diagram and in the fault tree. |
| **Stereotype** | Free text shown as «stereotype», e.g. `sensor`, `power`. |
| **Failure model** | *Constant failure rate λ* (per hour) or *Fixed probability* (per mission). Switching converts the value so the probability stays the same. |
| **Value** | λ ≥ 0, or a probability from 0 to 1. The line below shows the resulting mission probability. |
| **Common-cause group** | Blocks with the same name form a CCF group. Shown only when **Model common-cause failures** is on. |
| **Shared failure event** | Blocks with the same tag are **one physical failure** drawn in several places. They share one basic event, counted once, and their failure data stays in sync. Shown as `≡ TAG` above the block. |
| **Never fails (P = 0)** | Sets a fixed probability of 0. The block stays in the diagram and tree (dimmed) but never fails by itself. |

### Connection settings

| Field | Notes |
|---|---|
| **Flow** | What passes through, e.g. `28V`, `telemetry`. Documentation only. |
| **Direction** | **One way** (A → B: B needs A) or **Two way** (A ↔ B). |
| **Does B need A's flow?** | **Yes, required** (the default): B loses its output if it loses this flow. **Yes, as a backup**: the flow joins a redundancy group at B. **No** (two-way only): B doesn't depend on it. |
| **Does A need B's flow?** | The same question for the other end of a two-way connection. |
| **Redundancy group** | Connections into the same block with the same group name back each other up. |
| **Group loss threshold (k)** | Appears when a group has 2 or more inputs. The group is lost when **at least k** of its n inputs are lost: k = n (default) is AND, k = 1 is OR, anything between is a vote. The label shows `k/n` when k < n. |
| **Reverse direction** | One-way only: swaps source and target. |

**Reading the labels:**
- **Backups are dashed:** a connection with a backup at either end is drawn dashed.
- **Two-way connections** have an arrowhead at each end. Their label shows the non-default answers: `→` for the target's need, `←` for the source's, `∥group` for a backup and `✕` for not needed.
- **Switching back to One way** keeps whichever direction is still needed, reversing the arrow if necessary.

**For a backup to work as intended**, put the block's main input in the **same** redundancy group as the backup. Otherwise the main input stays required and the backup never comes into play.

### Frames

A frame is a labelled box around blocks, such as a subsystem or a redundant channel.

- **Effect on the tree:** where a block outside the frame depends on blocks inside it, the failures coming out of the frame are gathered under a gate with the frame's label, drawn as a blue box.
- **Membership:** a block is inside a frame when its centre is. The frame panel lists its blocks.
- **Gate type:** inside a required input or an AND group, the frame gate has the same type as the gate it sits in, so the logic is unchanged. Inside a k-of-n group each input is labelled separately, because a vote can't be split.
- **Nesting:** frames can be nested.
- **No effect on results:** frames only regroup the tree. Probabilities and cut sets are the same with or without them.

**Combined probability.** Tick **Show combined probability** to show the subsystem's failure probability in an orange tag at the frame's top-right corner, or just above it if the frame is narrow.
- **What it counts:** only failures of the blocks inside the frame. Blocks outside are taken as working, so the number describes the subsystem itself.
- **Internal structure counts:** redundancy, voting, shared failure events and common-cause failures inside the frame are all included.
- **The subsystem is lost when:**
  - **All of its outputs are lost** (the default). The outputs are the blocks inside that feed a block outside.
  - **Any of its outputs is lost**, for a frame that delivers several separate functions.
  - A frame that feeds nothing outside treats every block inside as an output.

It doesn't change the fault tree or any analysis.

### Settings

| Field | Notes |
|---|---|
| **Mission time (hours)** | Converts failure rates to probabilities: P = 1 − e^(−λT). |
| **Show failure data on blocks** | Hides the λ and P values on the diagram for a clean architecture view. The data is still used. |
| **Model common-cause failures** | Shows the common-cause controls and includes the groups in the results. Turning it off keeps the groups but ignores them, and the note says how many blocks still have one. |

---

## Analyses

An analysis names an undesired event and says which blocks must lose their output for it to happen. The diagram is the same for every analysis; only the fault tree changes.

| Field | Notes |
|---|---|
| **Showing** | The analysis whose results are shown, listed alphabetically. The fault tree exports follow it. |
| **+ New analysis**, **Duplicate**, **Delete** | Manage analyses. There's always at least one. |
| **Undesired event** | The label at the top of the fault tree. |
| **It occurs when** | **any** of the listed items occurs (OR), **all** of them do (AND), or **at least k** of them do. |
| **Include** | A shortcut that sets every block in the list at once to **Everything** or **Own only**. It shows **a mix** when the rows differ. |
| Outline | The items beneath the top event, as an indented tree: blocks, other analyses, event groups, basic events and undeveloped events (see [Building the tree top-down](#building-the-tree-top-down)). Click an item's name to edit it in the editor under the canvas. **↑** **↓** reorder items within their group (the order is the left-to-right order in the tree), and **✕** removes one. |
| **+ Add a block or analysis…**, **Pick on diagram** | Add blocks or other analyses. With Pick on diagram, tap blocks to add or remove them; they're outlined while you pick. Press **Done** or **Esc** to finish. |
| **+ Event group**, **+ Basic event**, **+ Undeveloped** | Add the top-down items described below. |

**Add to** sets where the panel's add buttons and **Pick on diagram** put new items:
- **Top event** (the default), or any event group in the analysis, shown indented.
- **Selecting or creating an item never changes it.** So after you create Event group 1, the next item still goes under the top event, beside it.
- **To fill a group,** choose it in **Add to**, or select the group and use its own add buttons in the editor under the canvas.
- **If the chosen group is removed,** Add to goes back to the top event.

New groups are numbered automatically: Event group 1, Event group 2 and so on.

### Building the tree top-down

The outline is a fault tree you build yourself, from the top event down. It stays linked to the diagram, so block failure data on the canvas always flows into it.

| Item | What it is | Drawn as |
|---|---|---|
| **Block** | A block on the diagram, set to **Everything** or **Own only**. | Its own gate, or a circle |
| **Analysis** (▣) | Another analysis, with its own settings. | An orange-outlined event |
| **Event group** (⬠) | An intermediate event you name, with its own gate: any, all or at least k of its causes. Groups can be nested to any depth. | A box and a gate |
| **Basic event** (○) | A cause that isn't on the diagram, such as a software fault, operator error or RF interference, with its own probability per mission. | A circle |
| **Undeveloped event** (◇) | A cause you estimate now and break down later. | A diamond |

**Editing.** Click an item's name in the outline, or **its box in the fault tree**, to open it in the editor under the canvas.

| Selected | What you can do |
|---|---|
| **Event group** | Rename it, choose its gate and k, reorder or remove its causes, and add blocks, analyses, groups and events directly inside it. |
| **Basic or undeveloped event** | Rename it, set its probability, switch between basic and undeveloped, or **Develop into an event group** to break it down. |
| **Block** | Choose Everything or Own only, **Develop one level**, or select the block on the diagram. |
| **Analysis** | Show it, to edit its own tree. |

Any item can be removed with **Remove from analysis** or the **Delete** key. Undo covers every change.

**Develop one level.** On a block set to Everything, **Develop one level** replaces the block with an event group of its causes from the diagram:
- the block's own failure;
- each required input;
- each redundancy group, as its own group with the right gate.

The result is unchanged; it's the same logic, written out one level. You can then rename, regroup, delete or develop further. For example, developing the Flight Controller gives "Flight Ctrl output lost" = any of: Flight Ctrl (own only), PDB, ESC, "All IMU inputs lost" and "All GNSS inputs lost".

**Example.** "Loss of nav solution" = **all** of:
- "Computer unavailable" = any of: Flight Ctrl (own only), Software fault (basic event, 0.01);
- "Attitude sources lost" = at least 2 of: IMU A (own only), IMU B (own only), Icing (undeveloped, 0.05).

The tree reads exactly as written. The result, 5.0 × 10⁻⁸, matches the hand calculation.

**Calculation.** Basic and undeveloped events join the same exact calculation as blocks:
- each one is counted once, wherever it appears;
- an event group with no causes never occurs;
- frames don't add labels directly inside an event group, because the group is your own structure.

### Everything or own only, per block

Each block in the list has its own setting:

| Setting | Counts |
|---|---|
| **Everything** (default) | The block's own failure, plus anything upstream that stops it working, such as lost power or input data. |
| **Own only** | Only the block's own failure. Its inputs are ignored. |

Settings can be mixed in one analysis. For example, "any of: Flight Ctrl (own only), ESC (everything)" means the Flight Controller fails itself, or the ESC stops working for any reason.

- **What own only still includes:** shared failure events and common-cause failures between own-only blocks, because those are failures of the blocks themselves.
- **Own failure plus only *some* inputs:** list the block as own only, together with the input blocks you want. For example, "Flight Ctrl fails or loses power" is any of Flight Ctrl (own only) and PDB (everything).
- **This is exhaustive.** Each block offers two events, "fails itself" and "stops working for any reason". Any, all and at least k, with analyses included in other analyses, can express any combination of them.

### Analyses inside analyses

The *Other analyses* section of **+ Add a block or analysis…** lists analyses that can be included. An included analysis is marked ▣ in the list. In the tree it appears as an intermediate event with an orange outline, with its own causes beneath it. It keeps its own settings.

- **Shared causes are counted once.** In the example, losing flight control almost always means losing thrust too, because the ESC needs the Flight Controller. So an analysis "Mission loss" = any of "Loss of thrust" and "Loss of flight control" is 7.93 × 10⁻⁵, not the naive sum of 1.59 × 10⁻⁴. The result satisfies P(A or B) = P(A) + P(B) − P(A and B) exactly.
- **No loops:** an analysis can't include itself, directly or through another, so those analyses aren't offered.
- **Deleting an analysis** removes it from every analysis that included it, and Undo brings it back.

---

## Results

| Section | What it shows |
|---|---|
| **Probability** | P(undesired event) per mission, labelled *exact* (or *rare-event approx.* for very large models). |
| **Occurs about once in** | 1 / P missions. |
| **Minimal cut sets** | Count of the smallest event combinations that cause the event. |
| **Single-point failures** | Events that cause it on their own. |
| **Repeated events** | Basic events drawn under more than one branch of the tree. Each is counted once. |
| **Fault tree** | See [The fault tree](#the-fault-tree). |
| **Minimal cut sets table** | Each set with its order (SPF, CCF, ×2, ×3…), probability and share of the total. |
| **Component importance** | Fussell–Vesely: the share of P that disappears if the event can't occur. |

### The fault tree

- **Layout:** the undesired event is at the top, then the gates and basic events beneath it.
  - **Gates:** OR and AND gates, and voting gates (OR shape marked `k/n`). Every gate shows its exact probability.
  - **Basic events** are circles.
  - **Boxes** grow taller for long descriptions, and every box in a row matches the tallest.
  - **Blue boxes** are frame labels; **orange-outlined boxes** are included analyses.
- **Click to edit:** boxes that stand for an item in your analysis (blocks, event groups, events) can be clicked to open that item in the editor.
- **Undeveloped events** are drawn as diamonds (◇).
- **No one-input gates.** A gate with a single input does nothing, so it isn't drawn: the box connects straight to that input. For example, an analysis on one block set to own only shows the undesired event directly above that block's failure.
- **Transfers.** A branch that's needed in several places is drawn in full once and tagged **T1**, **T2**…. Every other place shows a △ with the same label. It's one event, counted once.
- **Repeated basic events** have a heavier circle and an `R×n` tag.
- **Colours:** red = single-point failure; dashed circle = common-cause event; dimmed = probability 0.

**An AND gate is not the product of its inputs when they share a cause.** If both inputs depend on the same component, the gate occurs whenever that component fails, so its probability is much higher than the product. See [Finding a single point of failure](#finding-a-single-point-of-failure).

**Zoom.** The tree starts at **Fit**, showing the whole tree without scrolling. Hover over a box for its full label and probability.

| Control | Action |
|---|---|
| **+** / **−** | Zoom in or out around the centre of the panel. |
| **Ctrl + mouse wheel**, or a trackpad pinch | Zoom around the pointer. A plain wheel scrolls the page. |
| Drag | Pan when zoomed in. |
| **Fit** | Back to the whole tree. |

The zoom stays put while you edit. Exports are always full size.

---

## How the analysis works

### Block logic

A block's output is lost if **any** of these happen:

- the block fails internally;
- a **required** input is lost;
- **at least k of the n** inputs in a redundancy group are lost (k = n by default).

An analysis's undesired event is lost by the same rules, with the listed items as its inputs and no failure of its own. A block set to **own only** counts as lost only when it fails internally.

### Probabilities

- Constant failure rate: **P = 1 − e^(−λT)**, where T is the mission time.
- Fixed probability: used as is.
- Basic events are independent, apart from common-cause events.

### Repeated events: exact, counted once

A basic event is *repeated* when it reaches the undesired event through more than one path. That happens when a block feeds several others, when a CCF event is shared by several members, when blocks share a failure tag, or when two included analyses have causes in common. Multiplying and adding gate probabilities naively would count such an event more than once.

The page builds a **binary decision diagram (BDD)** of the whole logic, where every basic event is one variable. The probability, every gate probability in the tree and the importance values are computed exactly from it.

Example: A feeds both B and C, and the event needs both B and C lost. With P(A) = 0.1, P(B) = 0.2 and P(C) = 0.3:

| Method | Result |
|---|---|
| Exact | 0.1 + 0.9 × 0.2 × 0.3 = **0.154** |
| Treating the branches as independent | 0.193 |

If a model is too large for the BDD (more than 300,000 nodes), P falls back to the **rare-event approximation**, the sum of cut-set probabilities, which overestimates. The readout says so.

### Loops

Two-way connections, and one-way connections that form a cycle, create loops. They're calculated exactly: a loss must start in a real component failure and then spreads around the loop. A loop on its own can't keep a failure going.

- **Probability:** loops are found automatically and solved by repeating until nothing changes (the *least fixed point*).
- **Tree:** it's still an ordinary acyclic fault tree. A branch that would lead back to a block already above it on the same path stops there.
- **Checked:** across many random architectures with loops, two-way links and per-end settings, the result matched an exhaustive check of every failure combination.

### Minimal cut sets

Minimal cut sets are read straight from the BDD, so they're exact. A k-of-n group produces every k-member combination; 2 of 6 gives C(6,2) = 15. Events with probability 0 are left out. The table shows up to 20,000 sets.

### Common-cause failures (alpha-factor model)

For a group of m members with alpha factors α₁ … αₘ:

- αₜ = Σ k·αₖ
- Qₜ = the average of the members' probabilities
- Qₖ = k / C(m−1, k−1) · αₖ / αₜ · Qₜ, the probability of each **specific** event that fails exactly k members together

Each member's own event becomes its **independent** part, α₁ / αₜ × P(member). One CCF event is added for every combination of 2 or more members; a group of 3 gets three pair events and one triple. A member fails if its independent event or any CCF event that includes it occurs.

The panel lists each group with editable **α₁ … αₘ** and the resulting Qₜ, Q₁ … Qₘ. Values that don't add up to 1 are normalised, with a warning.

---

## Modelling guidance

### Shared components

Each block is one basic event, so a component that several others depend on is counted once. There are two ways to draw one:

- **Draw it once and connect it everywhere it's needed.** No other setting is needed.
- **Draw it in several places and tag the copies** with the same **shared failure event** tag, such as `ANT-1`. Use this when one block can't reasonably connect to everything. The copies become one event, and their failure data stays in sync.

Keep these cases apart:

- **Untagged blocks are always separate components**, even with the same name. Pasting an untagged block models a second, independent part.
- **Identical parts that can fail together** from a shared cause, such as the same batch, environment or software, are separate blocks in a common-cause group, not a shared failure event.

### Two-way connections

Set each end to what that block really needs:

| Situation | Setting |
|---|---|
| Each end can't work without the other (handshake, closed control loop) | Required both ways |
| Data flows both ways, but one side carries on without the replies | **No** at that side, or a one-way arrow |
| Cross-channel or backup data between redundant units | **Backup** at each end, in the same group as that unit's main input |

**Avoid "required both ways" by default.** It makes everything on one side depend on everything on the other. For example, if a computer and a radio are required both ways, losing the radio's ground link can count as losing the computer. When an analysis on one block lists unrelated parts of the system as single-point failures, check its two-way connections first.

### Narrowing an analysis without changing the diagram

To ask a narrower question, such as "what stops the autonomy computer itself?":

| Situation | Use |
|---|---|
| The dependency is real, but you want to leave it out of this one question | Set the block to **Own only** and add only the inputs you want, each set to Everything or Own only |
| The dependency isn't real, e.g. data passes through but its loss doesn't stop the block | Correct the connection: set that end to **No** |
| You want to leave a part out of every analysis | Correct or remove its connections. Don't set its probability to 0 |

Change a connection's **end** setting rather than reversing its arrow. Reversing a two-way connection changes nothing, and reversing a one-way arrow makes the *other* block depend on this one.

---

## Worked examples

### Comparing with a hand calculation

**The equation.** A tree expands to:

```
[(A·B) + (C + B + D + E)] · [(B + F + G) + G] · H
```

**Hand simplification.**

1. Absorption, B + A·B = B, gives B + C + D + E.
2. Idempotence, G + G = G, gives B + F + G.
3. Distribution, (X + Y)·(X + Z) = X + Y·Z, gives:

```
[B + (C + D + E)·(F + G)] · H
```

**What the page does instead.** It applies the same laws while building the minimal cut sets: each event appears once in a cut set, and supersets are removed. The 7 minimal cut sets are {B, H}, {C, F, H}, {C, G, H}, {D, F, H}, {D, G, H}, {E, F, H} and {E, G, H}.

**Probabilities used:**

| Event | A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|---|
| P | 2.56E−06 | 1.10E−03 | 1.16E−03 | 5.36E−06 | 1.20E−03 | 1.10E−03 | 1.10E−02 | 7.77E−04 |

| Method | P |
|---|---|
| Hand calculation, + as addition (rare-event approximation) | 8.7694 × 10⁻⁷ |
| **Exact (BDD)** | **8.7688 × 10⁻⁷** |
| Brute force over all 256 combinations | 8.7688 × 10⁻⁷ |
| Unsimplified equation with + and · applied directly | 6.5 × 10⁻⁸ (wrong) |

- **The hand result is right to 3 significant figures.** Treating + as addition slightly overestimates; the exact rule is P(X or Y) = P(X) + P(Y) − P(X)·P(Y).
- **The unsimplified equation is 13 times too low** because it multiplies the two B terms as if they were independent failures. The page avoids this automatically.

### Finding a single point of failure

A UAV's control link is lost only if **both** links are lost:
- **Connect 5G** = any of: 5G radio, Single Board Computer (SBC), antennas.
- **Connect SL** = any of: SBC, SL radio, Rx and Tx antennas.

Every block is set to own only, and the SBC appears in both links.

**Simplify:** (S + c + x) · (S + r + rx + tx) = **S + (c + x)·(r + rx + tx)**

So the link is lost if the SBC fails, or if both a 5G-only and an SL-only part fail.

| Quantity | Value |
|---|---|
| P(SBC) | 1.1 × 10⁻³ |
| P(5G-only part) · P(SL-only part) | 4.50 × 10⁻⁴ × 5.31 × 10⁻⁴ |
| **P(control link lost)** | S + (1 − S) × 2.39 × 10⁻⁷ = **1.100 × 10⁻³** |
| Share from the cut set {SBC} | 99.98% |
| P if the SBC never failed | 2.39 × 10⁻⁷, about 4,600 times lower |
| Naive product of the two links | 2.53 × 10⁻⁶ (435 times too low) |

The page gives the same numbers. It lists the SBC as the only single-point failure, with 99.98% importance, and marks it `R×2` in the tree. The two radios back each other up, but not against the computer they share.

---

## Import and export

| Export | File | Contents |
|---|---|---|
| Architecture → **PowerPoint** | `architecture.pptx` | One widescreen slide built from editable shapes: blocks, frames with probability tags, connections with arrowheads (routed paths included), ports, flow labels and tags. |
| Architecture → **SVG** | `architecture.svg` | A vector image of the diagram, cropped to its content. |
| Architecture → **SysML v2** | `architecture.sysml` | Textual notation (see below). |
| Fault tree → **PowerPoint** | `fault-tree.pptx` | The tree of the analysis being shown, as editable shapes, with the probability in the title. Large trees are scaled to PowerPoint's 56-inch slide limit. |
| Fault tree → **SVG** | `fault-tree.svg` | A vector image of the tree. |
| Fault tree → **Open-PSA XML** | `fault-tree.xml` | The tree in the Open-PSA Model Exchange Format (see below). |

**Saving the files:**
- **Opened directly in a browser:** files download straight away.
- **Inside Claude:** each export asks you to confirm. Two formats are saved with an extra `.txt` ending there (`architecture.sysml.txt`, `fault-tree.opsa.xml.txt`), so rename them after saving.
- **PowerPoint export** loads the PptxGenJS library from cdn.jsdelivr.net the first time it's used, so it needs an internet connection.

**SysML v2** contains:
- one `part def` per block, with its failure data, common-cause group and shared failure event;
- ports per connection, and a `part system` with every part and connection;
- `redundancyGroup` on redundant connections;
- `bidirectional`, `targetNeedsSource` and `sourceNeedsTarget` on two-way connections;
- the mission time, and the analyses as a comment.

The file carries the full project in a comment (`ft-layout:…`), so **Load file…** restores it exactly. Only SysML exported from this page can be loaded back.

**Open-PSA XML** is the exact tree on screen:
- **Gates:** one per gate, including block, group, frame and analysis gates, each with its label.
- **Basic events:** one per basic event, with per-mission probabilities, already converted from rates and split by the alpha-factor model.
- **k-of-n groups** become `<atleast min="k">`.
- **Page-only details** go in `<attributes>`, which other tools ignore.

Open the file in the [Fault Tree Editor](#fault-tree-editor), or check the results with another Open-PSA tool such as SCRAM.

**Browser storage.** The current project is kept in your browser automatically (key `ft-architecture-v1`), on that browser only. Save a `.json` for anything you want to keep or share.

---

## Project file format

```json
{
  "T": 1,
  "ccfOn": false,
  "showData": true,
  "active": "a1",
  "analyses": [
    { "id": "a1", "name": "Loss of thrust", "mode": "atleast", "vote": { "Targets": 2 },
      "targets": [ { "id": "m1", "group": "Targets", "own": false } ] }
  ],
  "blocks": [
    { "id": "m1", "name": "Motor 1", "st": "actuation", "mode": "rate", "v": 2e-5,
      "ccf": "", "rep": "", "x": 885, "y": 12 }
  ],
  "links": [
    { "id": "l9", "from": "fc", "to": "esc", "flow": "PWM ⇄ telemetry",
      "both": true, "needB": "req", "needA": "req", "group": "", "groupA": "" }
  ],
  "frames": [
    { "id": "fimu", "label": "Loss of attitude data", "x": 344, "y": 16, "w": 160, "h": 184,
      "showP": true, "lost": "all" }
  ],
  "ccf": {},
  "vote": {}
}
```

| Field | Type | Meaning |
|---|---|---|
| `T` | number > 0 | Mission time in hours. |
| `ccfOn` | boolean | Whether common-cause groups are modelled. |
| `showData` | boolean | Whether failure data is shown on blocks. |
| `active` | string | Id of the analysis being shown. |
| `analyses[].name` | string | The undesired event. |
| `analyses[].mode` | `any`, `all` or `atleast` | How the items combine. |
| `analyses[].vote` | object | `{"Targets": k}` for at least k. |
| `analyses[].targets[]` | array | The items under the top event. Every item has a `uid`. A block item has `id` (block id) and `own`; an analysis item has `id` = `"@"` + its id. An event group has `kind: "gate"`, `label`, `mode` (`any`, `all`, `atleast`), `k` and `children` (more items). A basic or undeveloped event has `kind: "event"` or `"undev"`, `label` and `p`. Top-level items have `group` = `"Targets"` for all / at least k. |
| `blocks[].name` / `st` | string | Name and stereotype. |
| `blocks[].mode` / `v` | `rate` or `prob` / number | λ per hour, or probability 0–1. |
| `blocks[].ccf` / `rep` | string | Common-cause group and shared failure event tag, or empty. |
| `blocks[].x`, `y` | number | Position on the canvas. |
| `links[].from` / `to` | string | Source and target block ids. |
| `links[].group` | string | Redundancy group at the target, or empty for required. |
| `links[].both` | boolean | `true` for a two-way connection. |
| `links[].needB`, `needA` | `req`, `grp` or `no` | Two-way only: does the target need the source (`needB`), and does the source need the target (`needA`)? |
| `links[].groupA` | string | Two-way only: backup group at the source. |
| `frames[]` | array | `label`, position and size, `showP` (show combined probability) and `lost` (`all` or `any`). |
| `ccf` | object | CCF group name → alpha values α₁ … αₘ. |
| `vote` | object | `"<block id>|<group>"` → k, for redundancy groups on connections. |
| `cw`, `ch` | number | Canvas size when it's larger than 1300 × 520. |

---

## Getting failure data

| Parameter | Typical source |
|---|---|
| Component failure rates λ | **NPRD-2023** (nonelectronic parts), MIL-HDBK-217 or similar for electronics, supplier data, your own field data. |
| Failure mode split (lost vs wrong output) | **FMD-2016**, Quanterion's companion to NPRD. Not modelled in this version. |
| Alpha factors | Your own maintenance and incident records; US NRC common-cause parameter estimates (NUREG/CR-5497) by analogous component type; or a beta factor from **IEC 61508-6 Annex D**. |

- **Alpha factors from your own records:** count the failure events that hit exactly k members of the group, nₖ. Then αₖ = nₖ / Σ nᵢ.
- **From a beta factor (pairs only):** α₂ = β / (2 − β) and α₁ = 1 − α₂. For example, β = 5% gives α₁ ≈ 0.974 and α₂ ≈ 0.026. For groups of three or more, there's no unique conversion from a single β.
- **Don't confuse NPRD's α and β with CCF factors.** In NPRD they're the Weibull characteristic life and shape parameters.

---

## Fault Tree Editor

`Fault Tree Editor.html` edits a fault tree directly, without the diagram behind it. Use it to adjust a tree for a report, try what-if probabilities, or rework a tree from another Open-PSA tool. It recalculates exactly as you edit.

**Opening a tree:**
1. In the workbench, click **Fault tree → Open-PSA XML**.
2. In the editor, click **Open XML…**, or drop the file onto the page.

**Other files:**
- **Files from other Open-PSA tools** also open if they use OR, AND and at-least gates, with probabilities given as a `float`, a `parameter` or an `exponential` of two floats. Anything else is reported.
- **New tree** starts from one gate and one basic event.

**Editing.** Tap a box to select it.

| Selected | What you can do |
|---|---|
| **Top event or gate** | Edit the description. Change the type (OR, AND, at least k). Reorder (↑ ↓) or remove (✕) inputs. Add a new or existing event or gate. Tick inputs and click **Group checked under new gate** to add a labelled intermediate event. |
| **Basic event** | Edit the description and the probability per mission. |
| **Anything but the top** | **Insert gate above** wraps it in a new gate. **Remove from parent** takes it out of its gate. |

- **Basic events table:** every probability, sorted by importance, editable in place.
- **Undo and redo:** Ctrl+Z, and Ctrl+Shift+Z or Ctrl+Y.
- **Tidying up:** unused items are removed automatically, and loops can't be created.
- **Grouping keeps the result:** grouping inputs of an OR gate under a new OR gate, or of an AND gate under a new AND gate, adds a label without changing any probability.
- **Zoom:** the same controls as the workbench. Double-click a box, or press **Selection**, to zoom to it.
- **Exports:** Open-PSA XML, PowerPoint and SVG.

Edits in the editor don't update the workbench project, because the editor works on the tree, not the diagram. The tree is kept in the browser (key `ft-editor-v1`).

---

## Limitations

- **Not a certified tool.** Use it for early design and trade studies; verify safety-case numbers with a qualified tool and process (e.g. ARP4761, ISO 26262, IEC 61508).
- **One failure mode per block.** Every failure is "output lost". Erroneous or late output isn't distinguished, and flow types don't change the logic.
- **Static fault trees only.** No sequence dependencies, spares, repair or cross-monitoring.
- **Independence is assumed** apart from the common-cause groups you define. Shared hidden causes (software, environment, maintenance) must be modelled explicitly.
- **Common-cause groups** have 2–6 members, and Qₜ uses the members' average probability, which is most accurate when members are identical.
- **Constant failure rates**: wear-out and infant mortality aren't modelled.
- **k-of-n is count-based** and treats all inputs as equal. To capture specific bad combinations, such as two adjacent rotors, model those pairs explicitly.
- **Renaming a redundancy group** resets its threshold to k = n.
- **Frames use block centres**, so a block half inside a frame may or may not belong to it. Renaming a block changes its width and centre, which can move it in or out of a frame near the edge.
- **Overlapping blocks** can't be routed around; Tidy separates them.
- **Very large models** may fall back to the rare-event approximation, and the cut-set table shows at most 20,000 sets.
- **SysML import** reads only files exported from this page.
- **The Fault Tree Editor** supports OR, AND and at-least gates only (no NOT, XOR or house events), and its edits don't flow back to the workbench.

---

## References

- Quanterion Solutions, *Nonelectronic Parts Reliability Data (NPRD-2023)*.
- Quanterion Solutions, *Failure Mode/Mechanism Distributions (FMD-2016)*.
- US NRC, *Common-Cause Failure Parameter Estimations* (NUREG/CR-5497 and updates).
- IEC 61508-6:2010, Annex D, *Methodology for quantifying the effect of hardware-related common cause failures*.
- SAE ARP4761A, *Guidelines for Conducting the Safety Assessment Process on Civil Aircraft, Systems, and Equipment*.
- Open-PSA Model Exchange Format, https://open-psa.github.io
- OMG Systems Modeling Language (SysML) v2 specification.
