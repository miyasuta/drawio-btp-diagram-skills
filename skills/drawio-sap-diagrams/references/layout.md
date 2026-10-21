# Layout Guide

Defines the **implementation-level details** for generating draw.io XML (px sizes, coordinate calculations, edge connection points, margins). **Claude reads this file during Step 4 (draw.io XML generation).**

For logical placement rules (what goes where, how things combine), see [`../rules/conventions.md`](../rules/conventions.md). For individual element style definitions (BTP container, icons, etc.), see [`elements.md`](elements.md).

This file is the implementation layer that realizes the official SAP guidelines in draw.io. The source of truth is [`../GUIDELINES.md`](../GUIDELINES.md). If you find a discrepancy with the official guidelines, bring the `styles/` side in line with them (confirm with the user before changing anything).

---

## 1. Edge (Arrow) Routing and Labels

Covers edge shape (straight / bent), approach direction into a box, how to cross container boundaries, and label placement. Use this together with [§4](#4-aligning-connected-elements-making-straight-arrows-work) for how to align center coordinates.

### 1a. When to Use Straight vs. Orthogonal

> Logical principle, "align data flows in a straight line" → [`../rules/conventions.md` §3](../rules/conventions.md#3-align-data-flows-in-a-straight-line)

Draw connections between elements that are arranged in a straight line as a **straight (direct) arrow**. Using orthogonal when there's no need to bend introduces an unnatural bend, so avoid it. Only use **orthogonal (bent) lines** between elements that aren't arranged in a straight line.

### 1b. Forbid "L-Shaped Approaches" Into a Box (Prefer Straight Lines)

**Never create an L-shaped arrow** that extends horizontally (or vertically) from the source and then bends at a right angle to pierce a **non-facing edge** of the target box (entering the bottom/top edge despite coming from the side, or the left/right edge despite coming from above/below). Don't let the principle "orthogonal is allowed only between elements not arranged in a straight line" become an excuse for a lazy L-shaped approach. **First consider adjusting the position of the source so it can connect with a straight line.**

- ❌ Bad example: extending rightward from an element on the left, then bending upward just before the target to pierce the box's **bottom edge** (a line "coming in from the side and entering from the bottom")
- ✅ Good example: align the source's center Y with a point on the target's left edge, and pierce the **left edge with a horizontal straight line** (matching center coordinates as described in [§4 Axis Alignment](#4-aligning-connected-elements-making-straight-arrows-work))
- When a target has multiple inputs that can't all be aligned in a straight line, draw multiple straight lines entering **different heights/positions on the same edge** (keeping each individual line straight). If the target is a **box** rather than an icon, you can align multiple horizontal straight lines onto the same edge just by varying `entryY`
- Orthogonal is truly necessary only when neither the source nor the target can be moved, and they can't be aligned on facing edges (e.g., the §6a case of branching from a common root to multiple targets)

> Goal: keeping the arrow's entry and exit directions aligned to "the centers of facing edges" makes it easier for the reader to trace the line. An L-shaped approach makes it ambiguous "which edge does this enter from," and entering the bottom edge in particular tends to trigger the label-piercing problem described in §10 (upward-pointing arrows).

### 1c. For Edges That Cross a Container Boundary, Put Bend Points and Labels in a Clear Zone Outside the Border

An edge going from an element inside a container to an **element outside it** (e.g., a service inside BTP → OpenAI outside BTP) crosses the container's border. In this case:

- **Cross the border at a right angle, vertically or horizontally, exactly once.** Don't let the line cross the border diagonally, or run alongside it for a long stretch
- **Don't place the edge's bend point (a horizontal jog) or its label directly on top of the border line.** If a bend point or label overlaps the border (of the Subaccount / BTP, etc.), both the line and the text become unreadable. Push the bend point down into a **clear zone outside the border** (e.g., empty space below the container's bottom edge)
- To make that possible, **leave enough clear space (roughly 60–80px) between the container and the external target**, and place waypoints via `<Array as="points">` so the horizontal jog lands inside that zone (at least 15px away from the border line)
- When multiple edges pass through the same zone (e.g., two lines going from the bottom of BTP to OpenAI), offset each label's Y / X so they don't overlap

> Failure example (2026 CAP-RAG): for the edge from CI inside BTP → OpenAI outside BTP, orthogonal's automatic jog happened to land exactly on the Subaccount's bottom edge, so the `create embedding` label and the horizontal line overlapped the border and became unreadable. Fixed by moving the external target (OpenAI) down 45px to widen the clear zone, and using a waypoint to push the horizontal jog outside the border (40px below the border line).

> Related: the official Foundation Spacing rule, "spacing between objects should be even, roughly the height of the SAP logo" → [the Spacing section of `../GUIDELINES.md`](../GUIDELINES.md#spacing)

### 1d. Keep Edge Labels Off Bend Points, Centered on a Straight Segment; Align the Height of Parallel Labels

For an edge with a bend point (corner), **don't place its label on top of the bend point** (it hides the shape of the line, making the path unreadable). Also, floating it away from the line makes it **unclear which line it belongs to**. To avoid both:

- **Place the label on a straight segment** (a horizontal or vertical straight part). Avoid the bend point
- **Center the label on the line** (set `<mxPoint as="offset">` to `x=0;y=0`). Having the line pass through the label's center makes it instantly clear "this is that line's label." Don't float it away
- Specify the anchor position along the path using `mxGeometry`'s `relative="1"` + `x` (−1 = source ... +1 = target, 0 = midpoint). If you want it at a specific Y / X, back-calculate `x` from the ratio along the path length (`x = 2 × (distance from source / total path length) − 1`)
- **Align the Y (height) of multiple parallel/related edge labels.** Matching each edge's midpoint Y (for a straight line, `(source.y + target.y)/2`), or aligning them to the same Y via `relative`'s `x`, improves the overall readability of the diagram.

> Example (2026 CAP-RAG): the two labels `create embedding` (a bent line) and `query embedding + answer` (a straight line) were both placed at the center of each edge's vertical segment (so the line passes through the center of the text), aligned at Y=412.5. Since `create embedding`'s path length is 300 and we wanted it at a distance of 112.5 from the source (Y=412.5), `x = 2×(112.5/300) − 1 = −0.25`.

---

## 2. Container Size and Margins (Avoid Wasted Space, Align Child Elements)

Once the internal elements of a Subaccount or BTP container are placed, size the container to **fit what's actually needed**. Services consisting of just an icon + label are surprisingly small, so leaving a big box sized for the old layout makes the empty space stand out.

**Rules**:
- Once child element placement is settled, revisit the parent container's width / height
- Check for unneeded margins (especially on the bottom and right) in both directions
- After deleting or shrinking child elements, always shrink the parent container too
- **When there's only one child element, center it horizontally** (`child x = (container width - child width) / 2`). Vertically, center it within the available area (excluding the label region)
- **Match size and alignment across sibling service boxes at the same level**: for a horizontal arrangement, match **height + top Y**; for a vertical arrangement, match **width + left X**. Prioritize visual consistency even when the amount of internal content differs
  - ⚠️ **If you only align one edge (right or bottom) while widths (heights) differ, the opposite edge ends up misaligned.** Always check size and alignment together. Example (2026 CAP-RAG): Solace and API Client, stacked vertically on the left, had their right edges matching at 180, but widths of 140/145 and left edges of 40/35 didn't match, making them look uneven → fixed by unifying both to width 145, left edge 35

**Recommended margins inside a container**:
- Left/right: 30–55 px each (**left and right must always be equal**. If one side is wider or narrower, the inner container appears to drift to one side. Whenever you move a child element's position, always re-check that the parent container's left/right margins remain symmetric)
- Top: 40 px (see below when there's a label)
- Bottom: 40–55 px (if another container/icon (e.g. CIS) hangs below, the bottom can be wider accordingly. Top/bottom asymmetry is acceptable, but left/right must be symmetric)
- Horizontal spacing between child elements: around 100 px (roughly 170 px from one icon's left edge to the next icon's left edge)
- Vertical spacing between rows of child elements: **30–40 px from the bottom edge of the upper row's labels to the top edge of the lower row's icons** (base it on the label's bottom edge, not the icon's; 2-line labels extend 30–35 px below the icon). Looser than 40 px leaves wasted space; tighter than 30 px makes labels and the next row look crowded. Reference (2026 Dev/Target CI/CD diagram): row 1 icon center Y=120, row 2 icon center Y=240 (icon 50×50, 1-line label) → about 35 px gap. If an arrow runs vertically between the rows (e.g. a line from an upper-row element down to the lower row), leave enough room for it to be visible, and don't shrink below the lower bound

> Failure example (2026 external-ui): when Custom Domain was moved inside the Subaccount and the main flow shifted right, the BTP container ended up with an asymmetric left margin of 140px / right margin of 35px, making the Subaccount look shifted to the right. Fixed by adjusting BTP.x and width to make both margins 45px. **As a rule, the outer container's left/right margins should be symmetric.**

**Top margin for a labeled container**:

The label type affects where child elements start (relative to the container) along Y:

| Label type | Example | Child element start Y |
|---|---|---|
| SAP logo + title (separate cell) | BTP container | **90–110 px** |
| Embedded text label | Subaccount | **50–60 px** |
| Service icon + title (separate cell) | BWZ service box | **50–60 px** |

For an embedded text label (`spacingTop=10` + `fontSize=12`), the label's bottom edge sits at about y=28. Placing child elements at y=50 or later ensures **at least 20 px of gap** from the label.

**Watch for SAP service icon labels overflowing**:

The label of an SAP service icon (`verticalLabelPosition=bottom`) is drawn **outside** the icon's geometry (50×50). When sizing a container, base the calculation on the **label's bottom edge**, not the icon's bottom edge.

- **Downward**: a 2-line label (`fontSize=12`) extends roughly **30–35 px** below the icon's bottom edge. Leave at least 20 px of bottom margin below the label's bottom edge
- **Left/right**: the label is centered horizontally on the icon. If the label width exceeds the icon width (50 px), it overflows to the sides. For icons near the edge of the container, leave a side margin of at least **half the label width**

**Reference measurements** (a Subaccount with 2 services placed side by side):
- Subaccount: 330×210
- 2 SAP service icons inside (56×56), side by side at the same Y
- The whole BTP (Subaccount + a CIS container of 280×150 below it): 410×460

---

## 3. Standard Sizes

> Related: the official Icons rules (SAP BTP Service Icons must use the version with the grey background circle / Generic Icons are neutral grey) → [the Icons section of `../GUIDELINES.md`](../GUIDELINES.md#icons). Since the official text has no concrete px spec, the standard sizes below are project-specific operational values.

| Element | Standard size | Notes |
|---|---|---|
| SAP service icon (`mxgraph.sap.icon`, standalone) | **50×50** | For standalone placement, e.g. directly under a Subaccount |
| SAP service icon (identifying icon at the top-left of a box) | **32×32** | For placing an icon + title at the top-left of a service box (BWZ, CF, etc.). See ["Service Box With a Service Icon"](elements.md#service-box-with-a-service-icon) in `elements.md` |
| Generic Icon (`shape=image` + base64 SVG) | **40×40 (square)** | Use the table below for icons with a known aspect ratio. For unknown icons, place a temporary square and let the human adjust it later |
| SAP On-Premise / Cloud Solutions rectangle | 120×60 | |
| 3rd-party Identity Provider rectangle | 120×60 | |

**Known aspect ratios for Generic Icons**:

| Icon | Aspect ratio (w:h) | Placeholder size (based on width=40) |
|---|---|---|
| Application Clients | 28:23 | **40×33** |

Icons not covered above should get a temporary placement at 40×40 (square).

**Placing a Generic Icon placeholder**:
- Claude does not handle Generic Icon base64 data (see WORKFLOW.md)
- Claude places a **40×40 empty square** (`value=""`, `dashed=1` dashed border) as a temporary placeholder
- **Place the label as a child element of the placeholder** (`parent="<placeholder_id>"`). This way, moving the placeholder in draw.io moves the label with it (the same feel as an SAP service icon). Since the label's `value` lives on a separate cell rather than on the placeholder, when the human later swaps in the SVG image, the label doesn't disappear
- Example label text cell: `text;html=1;align=center;verticalAlign=middle;resizable=0;points=[];autosize=0;strokeColor=none;fillColor=none;fontSize=11;fontColor=default;fontFamily=Helvetica;`, placed with `parent="<placeholder_id>"`. Coordinates are **relative to the placeholder**: 4–6 px below the placeholder's bottom edge (e.g. `y="44"`), and if the label is wider than the placeholder, center it with a negative `x` (e.g. label width 90, placeholder width 40 → `x="-25"`)
- **Vertical arrows should originate from the label**: when a placeholder and its label are stacked vertically, the source of a vertically extending arrow should be the **label text cell** (see [§6](#6-specifying-edge-connection-and-exit-points)). Even though the label is a child element, it can still be specified as an edge's source/target
- For icons with a known aspect ratio, use the size from the table above for the temporary placement. For unknown icons, place a temporary square and let the human resize it when swapping the icon in
- What matters is **fixing the position first**. Even if the human fine-tunes the size after swapping in the real icon, the layout won't break as long as the center coordinates line up

---

## 4. Aligning Connected Elements (Making Straight Arrows Work)

A series of elements connected by an arrow must have their **center coordinates (X or Y) match exactly**. Even a 1 px offset in the center will cause draw.io to automatically bend it into an orthogonal line.

**Horizontal example** (same Y center = 210):
- `ac`: y=190, height=40 → center Y = 210
- `bwz`: y=182, height=56 → center Y = 210
- `tc`: y=182, height=56 → center Y = 210

**Vertical example** (same X center = 160):
- `eu`: x=140, width=40 → center X = 160
- `ac`: x=140, width=40 → center X = 160

To align elements of different sizes in a straight line, compute `y = center − height/2` (and likewise for X) to position them.

> Related: the official Lines / Connectors rules for direction and semantic colors → [the Lines / Connectors section of `../GUIDELINES.md`](../GUIDELINES.md#lines--connectors)

---

## 5. Implementation Details for CIS Placement

> Logical principle, "place CIS below the Subaccount" → [`../rules/conventions.md` §5](../rules/conventions.md#5-place-sap-cloud-identity-services-below-the-subaccount)

**Implementation rules**:
- Place the CIS icon (50×50) near the center of the Subaccount's bottom edge, in the lower section inside the BTP container
- The trust line between Subaccount and CIS becomes a short vertical line (Subaccount bottom-center → CIS top-center, pink, bidirectional)
- Size the BTP container's height to include bottom margin extending to CIS's **label bottom edge**

**BTP container height formula**:

```
BTP height = (SA's y + SA's height) − BTP's y   ← distance to SA's bottom edge
           + 30                                  ← gap between SA and CIS
           + 50                                  ← CIS icon height
           + 30                                  ← label height (estimate for 2 lines at fontSize=12)
           + 20                                  ← bottom margin
```

**Reference measurements** (the 2026-04-17-01 diagram):
- BTP: y=30, SA: y=80, h=300 → SA bottom edge=380
- CIS: y=410 (30px gap), icon bottom edge=460, label bottom edge ≈ 490
- BTP height = (380−30) + 30 + 50 + 30 + 20 = **480** → BTP bottom edge=510

---

## 6. Specifying Edge Connection and Exit Points

Covers making an arrow's connection points (entry / exit) explicit, and how to construct an exit point (the source side) that doesn't pierce an icon's label.

**Make the connection point (entry) explicit**: if you only specify an edge's source / target, draw.io decides the routing automatically, which can break the path when a container's position is adjusted. For important arrows, explicitly specify **`entryX` / `entryY`**.

| Goal | Setting |
|---|---|
| Pierce the target's top edge, centered | `entryX=0.5;entryY=0;entryDx=0;entryDy=0` |
| Pierce the target's bottom edge, centered | `entryX=0.5;entryY=1;entryDx=0;entryDy=0` |
| Pierce the target's left edge, centered | `entryX=0;entryY=0.5;entryDx=0;entryDy=0` |
| Pierce the target's right edge, centered | `entryX=1;entryY=0.5;entryDx=0;entryDy=0` |

**Usage examples**:
- TC → On-Premise / Cloud Solutions: `entryX=0;entryY=0.5` (target's left edge, centered)
- Subaccount ↔ CIS (trust): `exitX=0.5;exitY=1` → `entryX=0.5;entryY=0` (SA bottom-center → CIS top-center, a vertical straight line)
- Application Clients → CIS (authenticate): the source is AC's **label text cell** (not the placeholder). `exitX=0.5;exitY=1` → `entryX=0;entryY=0.5` (label bottom-center → CIS left-center, orthogonal)
- CIS → 3rd-party IdP (delegates auth): `entryX=0;entryY=0.5`

**Vertical arrows should originate from the label**: when a Generic Icon placeholder (shape) and its label (text cell) are stacked vertically, the source of a vertically extending arrow should be the **label text cell** (not the shape). Coming from the shape produces an unnatural arrow that jumps over the label.

- End User → Application Clients (vertical layout): source=`eu_label` (End User's label)
- Application Clients → CIS (authenticate): source=`ac_label`

The source side can likewise be specified with `exitX` / `exitY`. When multiple edges leave from the same edge of a shape, spread them out.

**When drawing a downward arrow from an SAP service icon (with an embedded label), separate the label into its own cell and use that as the source**: an SAP service icon (`mxgraph.sap.icon`) has its label **embedded in the icon cell** by default. Drawing a downward arrow straight from it makes the exit point the bottom edge of the geometry (= right above the label), so **the arrow pierces the label text**.

The reliable fix is to apply **the same pattern used for Generic Icons** (the "vertical arrows should originate from the label" approach described at the top of this section) to SAP icons as well:

1. Set the icon cell's `value=""` (remove the embedded label)
2. Place the label as an **independent text cell**, directly below the icon's bottom edge (`text;html=1;align=center;verticalAlign=top;...;fontSize=12;`, aligned to the icon's center X, `y = icon's bottom edge`)
3. Make the downward edge's **source that label cell**, exiting from `exitX=0.5;exitY=1` (the label's bottom-center)

This makes the label cell's bottom edge the exit point, so the arrow no longer pierces the label text. Horizontal edges (piercing a left/right edge) can still use the icon cell as source/target as before (only downward edges need to come from the label cell).

**Make the label cell slightly taller than the text, to leave a gap between the text and the arrow**: with `verticalAlign=top`, the text sits at the top of the cell, so making `height` **6–8px larger** than the actual text height (roughly 17px for one line) puts a bit of distance between the cell's bottom edge (= the exit point) and the text, so the label and arrow don't touch. Guideline: `height=24` for a 1-line label (17px text + 7px margin), around `height=38` for a 2-line label. No margin looks "too tight"; 12px or more looks "too much space" (these are field-verified appropriate values).

> ⚠️ **Don't offset downward from the icon's bottom edge using `exitDy`**: with an orthogonal edge, a **stub line** gets drawn from the perimeter to the offset point, and that stub still pierces the label — this doesn't solve the problem (verified). Always use the separate label cell approach above instead.

- **Upward-pointing arrows (piercing an icon from below) are forbidden as a rule per §10**. This rule is about keeping a downward arrow **leaving** an icon from piercing the label. Before concluding you really need a top label in your design, first consider whether separating the label into its own cell (as above) solves it

---

## MCP / Implementation Constraints

- You can't make something a container's child element via `parent_id` (a constraint of drawio-mcp-server) → overlay it using absolute coordinates instead
- `import-diagram` can be slow / sometimes times out → use `add-rectangle` / `edit-cell` instead, or have the human paste the XML by hand (see WORKFLOW.md)
- Making the logo larger than 37×19 stretches it horizontally, due to the effect of `imageAspect=0`
