---
name: drawio-sap-diagrams
description: Use when creating, modifying, or refining SAP BTP architecture diagrams in draw.io / drawio format. Triggers on requests like "describe this landscape", "draw a BTP architecture", "make a draw.io of <SAP services>", or any task producing diagram.drawio / .drawio files for SAP cloud solutions.
---

# drawio-sap-diagrams

A skill for creating SAP BTP architecture diagrams in draw.io, working together with Claude. The human describes the design in words, and Claude turns it into Mermaid → draw.io XML.

> **Conversational tone**: In this SKILL.md, "human" and "user" are used to describe the division of roles. When actually asking the user to do something in conversation, use a polite second-person imperative without an explicit subject (e.g., ❌ "The human should paste the XML into draw.io" → ✅ "Paste the XML into draw.io").

## Overview

Proceed through the three-step chain: words → Mermaid → draw.io XML.

- **Why put Mermaid in between**: draw.io XML is verbose and hard for a human to review. By first nailing down just the logical structure (nodes, edges, groupings) in Mermaid, you can focus purely on style when generating the draw.io XML.
- **XML is pasted by hand**: applying it to draw.io is done by the human, via copy-paste. This is faster than drawing directly through a tool.

---

## Workflow

```
1. Human: describe the design in words
      ↓
2. Claude: create a Mermaid diagram to visualize the logical structure (diagram.md)
      ↓
3. Human: review the logical structure in the Mermaid preview and give correction instructions
      ↓
4a. Claude: element inventory pass — create a text list of cells (ID / size / parent-child / edges)
      ↓
4b-1. Claude: logical coordinate pass — define each element's position with main-axis variables (main_axis_Y / GAP, etc.) and relative formulas
      ↓
4b-2. Claude: physical coordinate pass — fix the main-axis values in one place, evaluate the formulas, and write out the XML
      ↓
4.5 Claude: sap-diagram-review (structural review) — automated XML-based check
      ↑↓ if fixes are needed, iterate back to here
5. Human: manually paste the XML into draw.io
      ↓
5.5 Claude: sap-diagram-review (visual review) — visual check via screenshot
      ↑↓ if fixes are needed, go back to 4b-1 / 4b-2, then repeat 4.5 → 5 → 5.5
```

**Fix-loop rule**: if a problem is found at step 5 / 5.5, do **not** Edit ad hoc — always **go back to 4b-1 (logical coordinate pass)**. First sort out what changes (main-axis variables / relative formulas), then recompute the values at 4b-2, and re-run Step 4.5. This prevents partial fixes from breaking overall consistency.

---

## Output Location

The output location follows the instructions in the consuming project's `CLAUDE.md`. If unspecified, the skill suggests the following as the default:

```
diagrams/
├── diagram.md          ← Mermaid (logical structure, output of steps 2-3)
└── diagram.drawio      ← draw.io XML (output of step 4, for manual pasting)
```

If you want to keep multiple diagrams, customize this in the project's `CLAUDE.md` — for example, with date-based subdirectories.

---

## Step 1: Describe the Design in Words

The human explicitly tells Claude the following:

- **A list of components** (service names and roles)
- **Hierarchy / membership relationships** (what belongs inside what)
- **Relationships** (data flow, trust, authentication delegation, etc.)
- **SAP vs. non-SAP distinction** (affects color coding)

If anything is ambiguous, Claude asks numbered follow-up questions.

---

## Step 2-3: Verify the Logical Structure With Mermaid

Before writing Mermaid, Claude must always check the **Conventions** section below, and follow its composition patterns (pairing End User with Application Clients, the three-part on-premise connection set, where to place CIS, etc.), labeling conventions, and language rules.

The output goes to `diagram.md`. The human reviews it in VSCode's Mermaid preview and gives correction instructions in words if there are mistakes.

**Note**: don't decide style (colors, line types) in Mermaid. Focus purely on logical structure.

---

## Conventions (Applied From the Mermaid Stage)

**Design-level conventions** for how to combine diagram components, how to place them, and how to handle labels and language. These apply from the Mermaid (logical structure) stage onward. For draw.io appearance (colors, sizes, px coordinate values), see `references/layout.md` / `references/elements.md`.

### 1. Language

Unless otherwise instructed, **all** element names on the diagram should be in English.

### 2. Arrow Labels

Arrows should **generally not** be labeled. Only label relationships that carry special meaning, as follows:

| Label | Purpose |
|---|---|
| `trust` | The trust relationship between a Subaccount and Cloud Identity Services |
| `authenticate` | An authentication flow |
| `delegates auth` | Authentication delegation (CIS ↔ 3rd-party IdP) |

### 3. Align Data Flows in a Straight Line

Elements that are consecutive in a data flow or sequence should be **aligned in a straight line, whether horizontal or vertical**. Bends or staggering break the visual flow.

- **Horizontal example**: `Application Clients` → `SAP Build Work Zone` → `SAP Task Center` should be aligned at the same height
- **Vertical example**: a sequence flowing top to bottom should be aligned in the same column
- **Don't create an L-shaped approach**: bending a line that comes in from the side just before the target, so it enters the bottom/top edge ("coming in from the side and entering from the bottom"), is forbidden. Instead, adjust the position of the source so the line goes straight into the opposite edge of the target. Multiple inputs that can't be aligned should be drawn as multiple straight lines entering different points on the same edge

> For draw.io implementation details (when to use straight vs. orthogonal lines, banning L-shaped approaches, px-aligning center coordinates), see `references/layout.md` (§1a / §1b / §4).

### 4. Always Place End User and Application Clients as a Pair

Place **End User** and **Application Clients** (Generic Icon) **stacked vertically** as a pair, serving as the entry point into services inside BTP. End User on top, Application Clients below.

- End User → Application Clients → service inside BTP (e.g. Build Work Zone)
- **Authentication lines originate from Application Clients** (not from End User)

### 5. Place SAP Cloud Identity Services Below the Subaccount

SAP Cloud Identity Services (CIS) is an element inside the BTP container but outside the Subaccount. Its placement is fixed to **directly below the Subaccount (vertically)**.

- The trust line between Subaccount and CIS becomes a short vertical line
- Placing it to the left, right, or above makes it more likely to cross other elements, obscuring the meaning of the trust line

> For draw.io implementation details (px sizes, margins, computing the BTP container height), see `references/layout.md`.

### 6. Consolidate Arrows From a Service Box Into One

When a service box (e.g., Build Work Zone) contains multiple components (an app, Inbox, etc.), **don't draw arrows from each individual component to an external service — draw a single arrow from the outer box instead**.

- **Reason**: multiple arrows converging on one icon looks cluttered
- In Mermaid too, use `BWZ → SPA` rather than `UI5 App → SPA`

#### 6a. When a Service Box Fans Out to Multiple Targets, Align the "Root" to a Single Point

When a service box sends arrows to several different targets (e.g., HTML5 App Repository, XSUAA), **launch them all from the same single point on the box** (e.g., top-center, `exitX=0.5;exitY=0`), then branch out to each target using orthogonal routing.

- **Reason**: if the arrows' starting points are scattered across multiple spots on the box, it becomes hard to tell visually where they originate
- Specifying a common exit point lets draw.io's orthogonal router share the first segment, producing the look of a single vertical stem branching out
- Use orthogonal arrows rather than straight ones (since they go from a common root to different targets, they can't be straight lines)

### 7. Placing Applications Inside a Service Box

When placing **multiple applications** inside a service box such as Build Work Zone, **add an inner container (box)** and place a Generic Icon placeholder + text label side by side within it. If there is **only one** application, the inner container isn't needed — place the Generic Icon placeholder directly inside the service box.

```
┌─ BWZ ─────────────────────────┐
│ 🔵 SAP Build Work Zone        │
│ ┌───────────────────────────┐ │
│ │  [icon]      [icon]       │ │
│ │  App Name    My Inbox     │ │
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

- The inner container uses a blue border, white fill, rounded corners (the same style as the service box)
- Each app consists of a Generic Icon placeholder (40×40) + a text label below it
- Apps are laid out side by side with even spacing

> For draw.io implementation details (px sizes, styles), see `references/elements.md`.

### 8. On-Premise / Private Cloud Connection Pattern

When connecting from a BTP Subaccount to an on-premise or Private Cloud system, **place the following 3 services together as a set inside the Subaccount**:

```
Destination Service → Connectivity Service → Cloud Connector
```

- **Destination Service**: manages the destination configuration
- **Connectivity Service**: manages the tunnel to on-premise
- **Cloud Connector**: place it inside the box of the target system (outside BTP)

Cloud Connector is technically a BTP-side component, but on the diagram it's drawn **inside the box of the target system** (since it's physically installed in the on-premise environment).

### 9. Add the SAP Logo to SAP Product Boxes Outside BTP

For rectangles representing SAP products placed outside BTP (SAP S/4HANA, SAP SuccessFactors, etc.), **add the SAP logo + product name at the top left, following the same pattern as the BTP container**.

- Leave the rectangle's `value` empty, and display the product name via the logo cell (`image=img/lib/sap/SAP_Logo.svg`)
- This visually makes it clear that it is an SAP product

> For draw.io implementation details (font size, placement), see the "SAP Logo" section of `references/elements.md`.

### 10. Avoid Upward-Pointing Arrows as a Rule (Related to Label Placement)

The label of an SAP service icon (`mxgraph.sap.icon`) is drawn below the icon by default. As a result, **an arrow entering from below (an upward-pointing arrow) ends up piercing through the label**.

**Rule**: arrange arrows into an icon so they enter **from the side or from above** (i.e., don't use upward-pointing arrows). Adjust the position of the source side so the arrow direction stays downward or sideways.

- **Example**: when drawing an arrow from BWZ to a dependent service such as HTML5 App Repo / XSUAA, place the dependent service **below BWZ** (placing it above would create an upward arrow that collides with the label)
- **Reason**: if you switch the label position to above the icon to match an upward arrow, it ends up too close to the parent container's label (e.g., Subaccount), hurting readability. Also, mixing label positions above and below across the diagram breaks visual consistency

**Exception**: only when layout constraints make an upward-pointing arrow truly unavoidable, switch that icon's label to the top (`verticalLabelPosition=top;verticalAlign=bottom`). In that case, ensure **at least 20 px** of spacing between it and the parent container's label.

> For draw.io implementation details (style diffs for the label position variant), see the "Label Position Variants" section of `references/elements.md`.

### 11. Conventional Abbreviations for Service Names

For diagram labels, prefer **conventionally used abbreviations** over the official full name. Putting a long full name in as-is unnecessarily widens the diagram horizontally, so before resorting to a line break (see the label line-break rule in `elements.md`), first consider whether the name can be shortened.

Known abbreviations:

| Official name | Label on the diagram |
|---|---|
| SAP Destination service | `Destination` |

**General principles**:
- Suffixes like `service` / `Services` that aren't needed for identification can be omitted (but keep them where they're part of the brand, such as `SAP Cloud Identity Services`)
- The `SAP` prefix can also be omitted when there's no ambiguity in context (but keep it where it's needed to distinguish from something else, such as `SAP HANA`)
- When in doubt, keep the full official name, and only consider abbreviating it when it's crowding the horizontal space

If you adopt a new abbreviation, add it to this table.

---

### 12. Standardize External Entry Points Into BTP on the Left Side

External systems (event brokers, API clients, end users, external IdPs — anything coming from outside BTP into a service inside BTP) should be **grouped together on the left side of the BTP container**. If there are multiple entry points, **stack them vertically on the left**, with each arrow pointing rightward (into BTP).

- **Reason**: if entry points are scattered on all sides, it's no longer obvious at a glance "where does this enter the system." Aligning them in a left-to-right flow makes the origin of the data visually clear
- **Example**: place `Solace (event source)` on top and `API Client (question entry point)` below it, stacked vertically, with both sending rightward arrows into services inside BTP
- The End User + Application Clients pairing (§4) is also one application of this "entry on the left" principle

### 13. Don't Place Elements Above the BTP Container

As a rule, **don't place anything above the top edge of the BTP container**. External services outside BTP (external APIs such as OpenAI, non-SAP systems) should be placed to the **left, right, or below** BTP.

- **Reason**: placing things above BTP would require upward-pointing arrows from services inside BTP, which tend to pierce the SAP service icon's label below it (§10). It also throws off the visual balance of the whole diagram, making it harder to establish a reading flow of entry (left) → processing (inside BTP) → external integration (below)
- **Placement guide**: put entry-point-style externals on the left (§12), and external APIs called dependently by services inside BTP (e.g., an LLM) below — that way arrows from BTP can point naturally downward (from the bottom edge of a box-type service)
- **Exception**: if you truly must place something above, use a box-type service (one without a bottom label) as the connection point inside BTP, so the upward arrow doesn't pierce a label

### 14. Decide Container Membership (Inside BTP / Inside Subaccount / Outside) at the Mermaid Stage

Decide **which container each element belongs to** (① outside BTP / ② inside BTP but outside the Subaccount / ③ inside the Subaccount) **at the Mermaid (logical structure) stage**. If you realize "actually it belongs inside" only after you've started writing draw.io coordinates, it triggers a cascade of recalculating container widths, margins, and main-axis positions.

- **Make membership explicit with nested subgraphs in Mermaid** (place `subgraph SUB["Subaccount"]` inside `subgraph BTP`, and put each node at the correct level). If a node's membership is ambiguous, confirm it with the user during the Mermaid review
- **Decide a service's membership based on its provisioning scope**, not "where it appears in the request path." Judge it by "at what scope is this service entitled/configured," not by request routing
  - Example: **SAP Custom Domain service is configured at the subaccount level**, so even though it sits in front of the Approuter in the request path, on the diagram it goes **inside the Subaccount** (placing it directly under BTP, outside the Subaccount, would be wrong)
  - Example: XSUAA, Destination, HTML5 App Repo, API Management (Integration Suite), etc. are also subaccount-level services → inside the Subaccount
  - **For elements you're unsure about, do an inventory pass at the Mermaid stage and settle membership one by one before moving to 4a**

> Failure example (2026 external-ui): Custom Domain was initially placed directly under BTP (outside the Subaccount), but it was actually a subaccount-level service. Moving it inside the Subaccount required recalculating the main flow's X coordinate, the Subaccount width, and the BTP left/right margins. Settling membership at the Mermaid stage would have avoided all of that rework.

---

## Step 4: Generate draw.io XML (3-Pass Structure)

> **Required**: before entering 4a, you **must Read** `references/layout.md`, `references/elements.md`, and `references/coordinate-passes.md`. Writing from memory or guesswork risks missing size, coordinate, and SAPIcon-name rules that were updated in earlier iterations. If in doubt, also check `references/guidelines.md`.

Conventions were already applied in steps 2-3. Here, focus on draw.io-specific implementation (px sizes, coordinates, edge connection points, style values).

### The 3 Passes at a Glance

| Pass | Role | What it produces |
|---|---|---|
| **4a Inventory** | Catch structural mistakes early | A coordinate-free list of cells (ID / type / parent-child / edges) |
| **4b-1 Logical coordinates** | A position definition resilient to changes | Main-axis variables + relative formulas (e.g., `BWZ.x = SUB_LEFT + 35`) |
| **4b-2 Physical coordinates** | Fix px values and emit XML | Evaluated variable results + `<mxCell>` |

The procedure for each pass, typical examples of main-axis variables, how to write the formulas, and the rules for the fix loop are all gathered in [`references/coordinate-passes.md`](references/coordinate-passes.md). Always read it before executing Step 4.

### Step 4 Preparation (Identify Undefined Styles)

Before entering the 3 passes, do the following once.

1. **Inventory the elements you need**: list out the service icon names, box types, and any new layout patterns this diagram will use
2. **Check whether they already exist in references**: cross-check against the SAPIcon table in `references/elements.md`, the standard size table in `references/layout.md`, and `references/guidelines.md`
3. **Batch up unknowns for the user**: ask about unknown SAPIcon names or undefined patterns **all at once** (asking one at a time as you encounter them fragments the conversation)

---

## When You Encounter an Unfamiliar Style or Icon

If you encounter an element not defined in `references/` (an unregistered `SAPIcon` name, a new kind of rectangle, a color/line type used for the first time, etc.), **don't guess — check with the user**. Granularity of confirmation:

- **SAP service icons**: if the official `SAPIcon=...` name is unknown, **look it up first** following "Looking Up an Icon Name Not in the Table" in `references/elements.md`; ask the user only if that finds nothing. Don't guess the shape name (a wrong name renders as blank in draw.io)
- **Style specs**: for elements with no definition at the implementation layer (color, corner radius, line type, font, etc.), quote the relevant section of `references/guidelines.md` if one exists, otherwise ask the user
- **Layout decisions**: don't adopt a new placement pattern on your own (stacked containers, interface symbols, etc.)

Once confirmed, if you judge the result to be **a reusable rule, add it to the appropriate file**:

- **Composition patterns / placement rules** → the Conventions section of this SKILL.md
- **draw.io styles / sizes** → `references/layout.md` or `references/elements.md`

If the rule originates from the official guidelines, also add a cross-link to the relevant section of `references/guidelines.md`. This lets `references/` grow along with usage, so future sessions won't need to ask again.

---

## Division of Labor Between Claude and the Human

| Item | Who's responsible |
|---|---|
| Placing SAP service icons (`mxgraph.sap.icon`) | **Claude** |
| Placing Generic Icons (`shape=image` + base64 SVG) | **The human** (Claude only places an empty placeholder; see `references/layout.md` for sizes) |
| Containers, lines, text, styling in general | **Claude** |

**Why only Generic Icons are the human's job**: a base64-encoded SVG runs several KB per icon and has to be embedded in the `style` attribute, which blows up token usage and response time if Claude handles it in XML. SAP service icons are fine because they only need a short `SAPIcon=<name>` identifier.

Stating up front, at the start of the conversation, "Generic Icons will be placed by the human afterward, so please only write the SAP service icons into the XML" avoids misunderstandings.

### Instruction Tips

- **When you don't want icons touched**: "I'll handle swapping out the icons myself, so please only apply the other changes."
- **When you only want to change the style of an embedded label**: "Can you change just the text style while keeping it embedded in the container?" (this prevents it from being turned into a separate cell)
- **When you want to persist a new rule**: explicitly say, "Please add this to `references/layout.md`" (or `elements.md`)

---

## Step 4.5: Structural Review (Runs Automatically)

Right after generating `diagram.drawio` in step 4, Claude **automatically runs the sap-diagram-review skill (Phase 1: structural review)**. It checks the XML against the Conventions and the current rules in `references/`, and reports any problems it finds. If there are none, it displays "Review OK" and moves on to step 5.

## Step 5: Paste Into draw.io + Share a Screenshot

Paste the contents of the generated `diagram.drawio` into draw.io, and **attach a screenshot to the conversation**.

## Step 5.5: Visual Review (Runs Automatically)

Based on the screenshot the human shared, Claude runs the **sap-diagram-review skill (Phase 2: visual review)**. It checks the image for label overflow/overlap, container margins, arrow straightness, and so on, and reports any problems along with suggested fixes. If there are none, it displays "Visual review OK."

If there are problems, go back to giving correction instructions in words.

---

## References

| File | Role | When to read it |
|---|---|---|
| This SKILL.md (Conventions section) | Design conventions (composition patterns, labels, language) | **Steps 2-3** (Mermaid) |
| `references/layout.md` | draw.io implementation details (px sizes, coordinates, edge connection points) | **Step 4** (draw.io XML) |
| `references/elements.md` | draw.io element style definitions (BTP, logos, icons, lines) | **Step 4** (draw.io XML) |
| `references/guidelines.md` | English summary of the official SAP guidelines (source of truth) | Whenever you're unsure |

Before entering Step 4, you **must Read** `references/layout.md` and `references/elements.md`. Writing XML from memory or guesswork risks applying stale style values or coordinate rules.

---

## Common Pitfalls

| Failure | Cause | Countermeasure |
|---|---|---|
| Style information gets put into Mermaid during step 2 | Tried to decide logical structure and appearance at the same time | Mermaid is for logical structure only; style comes in step 4 |
| Redoing coordinate calculations every time a box size changes | Did the inventory and the coordinate calculations at the same time | Split step 4 into 4a (inventory) / 4b (coordinate calculation) |
| Claude changes styles on its own judgment | Didn't explicitly reference `references/` | State explicitly at the start of step 4: "follow the rules in `references/`" |
| A label ends up created as a separate cell | Claude interpreted "change the style" as "create a new cell" | Instruct: "keep it embedded in the container, just change the text style" |
| Claude touches the Generic Icon's base64 data | The division of labor wasn't communicated | State explicitly at the start of step 4: "the human will place the Generic Icons" |
| Claude forgets a rule it just decided on | It was agreed only in conversation, and never persisted | Instruct: "please add this to `references/layout.md`" (or `elements.md`) |
| The conversation gets fragmented confirming icon names | The user was asked about each unregistered icon as it was hit during XML generation | List the icons needed at the start of step 4, and confirm any unregistered ones **all at once** |
| Claude spends too much time on coordinate calculations | Tried to get pixel-perfect alignment | Final adjustments are the human's job. Claude should only be strict about the two endpoints of a straight arrow and membership relationships — everything else can come out rough |
