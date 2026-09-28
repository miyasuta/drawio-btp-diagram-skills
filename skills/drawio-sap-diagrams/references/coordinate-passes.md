# Step 4 Details: The 3-Pass Coordinate Structure

Details of the **3-pass procedure for generating draw.io XML**, referenced from Step 4 of `SKILL.md`. `SKILL.md` itself only holds the overview; the full mechanics are worked out here.

## The Big Picture

```
4a  Element inventory (write no coordinates)
    ↓
4b-1 Logical coordinates (define positions with main-axis variables + relative formulas)
    ↓
4b-2 Physical coordinates (fix the variables → evaluate the formulas → emit XML)
```

**Why split into 3 passes**: writing "cell structure," "logical coordinates (relative formulas)," and "physical coordinates (px values)" all at once means later fixes cascade into repeated rework.

- **4a (Inventory)**: catches structural mistakes early. Writing no coordinates at all lets you focus purely on the layout discussion
- **4b-1 (Logical coordinates)**: defines positional relationships with main-axis variables + relative formulas. Changes such as a label length change, moving an element, or shifting the main axis can then be handled with just a "formula change"
- **4b-2 (Physical coordinates)**: evaluates the formulas from 4b-1 to fix px values. For fixes, go back to 4b-1 and update the formulas

Example: if you define `Subaccount.h = (bottom of the lowest element's label) − Subaccount.y + MARGIN` as a formula, then a fix like "move branches from top to bottom" won't structurally create whitespace problems (a past failure case happened because, during a fix, the px values were edited directly instead of re-deriving the formulas).

---

## Step 4 Preparation (Identify Undefined Styles)

Before entering the 3 passes, do the following once.

1. **Inventory the elements you need**: list out the service icon names, box types, and any new layout patterns this diagram will use
2. **Check whether they already exist in references**: cross-check against the SAPIcon table in `references/elements.md`, the standard size table in `references/layout.md`, and `references/guidelines.md`
3. **Batch up unknowns for the user**: ask about unknown SAPIcon names or undefined patterns **all at once** (asking one at a time as you encounter them fragments the conversation)

---

## Step 4a: Element Inventory Pass (Write No Coordinates)

Before writing any XML, do a **coordinate-free** text inventory of all cells. For sizes, just quote the standard values from `references/layout.md` (don't calculate anything at this point).

Inventory items (output as a bulleted list):

- **Containers / boxes**: ID / type (BTP, Subaccount, service box, etc.) / size (standard value) / parent container
- **Icons / labels**: ID / type (SAPIcon name / Generic Icon placeholder) / parent container / label text
- **Edges**: ID / source → target / style type (straight / orthogonal / trust / authenticate, etc.)

It's fine to share this with the user at this point. Structural mistakes (wrong parent-child relationships, missing edges, wrong box type choices) are cheap to fix if caught here.

---

## Step 4b-1: Logical Coordinate Pass (Define With Variables and Relative Formulas)

Before writing the XML, express the coordinates as text in the form of **"main-axis variables" + "relative formulas."** Don't write raw px values directly.

Main-axis variables to define (typical examples):

- `main_axis_Y` — the center Y of the horizontal main flow
- `main_axis_X` — the center X of the vertical arrangement (for vertical flows)
- `H_GAP` — horizontal gap between elements (clearance from the edge of a label)
- `V_GAP` — vertical gap between elements
- `MARGIN_INNER` — inner margin inside a container
- `BTP_TOP` / `SUB_TOP`, etc. — the origin coordinate of each container

Write each element as a formula referencing the main axis / other elements:

```
BWZ.center_Y = main_axis_Y
BWZ.x        = SUB_LEFT + 35
Dest.x       = BWZ.right + H_GAP + dest_label.half_width - icon_half
CF.x         = Dest.right + H_GAP + dest_label.half_width
Subaccount.h = (HTML5.bottom + label_height) - Subaccount.y + MARGIN_INNER
BTP.h        = (CIS.bottom + cis_label_height + 20) - BTP.y
```

**Goal**: if a later fix comes in — "shift the main axis up," "widen the gap," "the label width changed" — changing a single variable propagates through the whole layout. This eliminates cascading manual recalculation.

---

## Step 4b-2: Physical Coordinate Pass (Fix the Variables and Emit the XML)

Assign concrete px values to the main-axis variables from 4b-1, evaluate the formulas, and write the XML.

Calculation order:

1. **Fix the main-axis variables**: decide `main_axis_Y` / `H_GAP` / container origins, etc.
2. **Evaluate the relative formulas**: plug values into the formulas from 4b-1 to compute each element's absolute coordinates
3. **Edge connection points**: for edges connected by a straight line, align both ends' cell center coordinates so they match (`references/layout.md` §4)
4. **Emit XML**: write out the `<mxCell>` elements

When a fix is needed (adding an element, a label width change, shifting the main axis), **go back to 4b-1 and update the formulas**, then re-run 4b-2. Resist the temptation to edit the XML directly.

---

## Coordinate Granularity

When Claude writes the XML, coordinates can be decided roughly. **Fine-grained final positioning is done by the human in draw.io.** The only things Claude needs to get exactly right are:

- **The center coordinates of elements connected by a straight arrow** (even a 1 px offset in the center will cause it to bend into an orthogonal line — see `references/layout.md` §4)
- **Containment relationships** (inside BTP / inside the Subaccount / inside a box — i.e., membership)

Everything else (container width/height, margins, arrow bend points, etc.) can be approximate. Rather than spending time getting pixel-perfect, it's faster to get rough XML out quickly and get feedback from the user.

---

## Source of Truth for Styling

- **`references/guidelines.md` = the English summary of SAP's official guidelines = source of truth**
- **`references/layout.md` / `references/elements.md` = the implementation layer that realizes the guidelines as draw.io XML**
- When the implementation layer and the official guidelines disagree, the implementation layer should generally be fixed to match the official guidelines. If Claude finds a discrepancy, it should not fix it unilaterally — **report it to the user first, then act** (since it may have been deliberately diverged in an earlier iteration).
