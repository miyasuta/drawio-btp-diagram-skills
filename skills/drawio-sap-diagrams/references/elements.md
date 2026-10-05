# Per-Element Style Definitions

A collection of style definitions for individual elements when drawing SAP BTP diagrams in draw.io. **Claude reads this file during Step 4 (draw.io XML generation).**

For layout principles (spacing, standard sizes, axis alignment, etc.), see [`layout.md`](layout.md).

This file is the implementation layer that realizes the official SAP guidelines in draw.io. The source of truth is [`../GUIDELINES.md`](../GUIDELINES.md). If you find a discrepancy with the official guidelines, bring the `styles/` side in line with them (confirm with the user before changing anything).

---

## BTP Container (Blue Rectangle)

> Related: the official Areas colors (SAP Border `#0070F2` / Fill `#EBF8FF`) and the fixed 16px corner radius → [the Areas section of `../GUIDELINES.md`](../GUIDELINES.md#areas-rectangular-zones). `strokeColor` / `fillColor` match the official spec. `arcSize=32` + `absoluteArcSize=1` results in an effective radius of 16 px per draw.io's spec, matching the official value (`arcSize` is treated as equivalent to a diameter).

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#EBF8FF;
       arcSize=32;absoluteArcSize=1;strokeWidth=1.5;verticalAlign=top;
       fontSize=14;fontStyle=1;fontColor=#002A86;
text: (empty)
```

- Leave the label empty; overlay the logo and title as separate cells

---

## SAP Logo

> Related: the official Product Names rule "SAP product names must be paired with the SAP logo / it's not recommended to use too many logos in one diagram" → [the Product Names section of `../GUIDELINES.md`](../GUIDELINES.md#product-names-product-name-labeling)

**⚠️ Required**: any container representing an SAP product (the BTP container, an SAP product box outside BTP) must have the SAP logo added. The logo cell also serves as the title text for the product name.

> Design rule: [`../rules/conventions.md` §9](../rules/conventions.md#9-add-the-sap-logo-to-sap-product-boxes-outside-btp)

```
style: image;image=img/lib/sap/SAP_Logo.svg;editableCssRules=.*;
       verticalLabelPosition=middle;labelBackgroundColor=none;verticalAlign=middle;
       imageAspect=0;labelPosition=right;align=left;
       fontSize=16;fontStyle=1;spacingLeft=0;fontColor=#002A86;spacing=0;
width: 37
height: 19
text: SAP BTP  ← also serves as the title text (displayed to the right of the logo)
```

- `labelPosition=right` makes the text appear to the right of the logo
- **37×19** is the correct aspect ratio for the size (going any larger stretches it horizontally)
- Place it in the top-left of the container (roughly container x+15, container y+11)
- Leave the BTP container rectangle's own `value` empty and overlay it with the logo cell
- The title text's style is baked into the logo cell; there's no separate standalone style defined for it

### The Logo in an SAP Product Box Outside BTP

Following the same approach as the BTP container, place the logo + product name at the top left of the rectangle. If the box is small, lower `fontSize` to `12`.

```
style: image;image=img/lib/sap/SAP_Logo.svg;editableCssRules=.*;
       verticalLabelPosition=middle;labelBackgroundColor=none;verticalAlign=middle;
       imageAspect=0;labelPosition=right;align=left;
       fontSize=12;fontStyle=1;spacingLeft=0;fontColor=#002A86;spacing=0;
width: 37
height: 19
text: S/4HANA Private Cloud  ← the product name (the "SAP" prefix can be omitted since the logo already represents SAP)
```

- The only difference from the BTP logo is `fontSize` (16 → 12)
- Placement is the same (container x+15, y+11)

### Title Text Style (Inside the Logo Cell)

| Property | Value |
|---|---|
| fontSize | 16 |
| fontStyle | 1 (bold) |
| fontColor | #002A86 |
| align | left |

---

## Subaccount Container (White Rectangle With a Blue Border)

> Related: the official Areas nesting rule "alternate between using a fill and not using a fill for contrast" → [the Areas section of `../GUIDELINES.md`](../GUIDELINES.md#areas-rectangular-zones). Placing a white Subaccount inside the BTP container (`#EBF8FF` fill) is an implementation of this alternate-fill rule.

A white rectangle representing a Subaccount, placed inside the BTP container.

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#ffffff;
       arcSize=32;absoluteArcSize=1;strokeWidth=1.5;
       verticalAlign=top;align=left;spacingLeft=20;spacingTop=10;
       fontSize=12;fontStyle=1;fontColor=default;fontFamily=Helvetica;
text: Subaccount
```

- The corner rounding matches the BTP container (arcSize=32)
- The "Subaccount" label is embedded in the container, aligned to the top-left
- The font is **12px bold** (`fontStyle=1`). This is one level below the BTP title (16px bold / SAP blue), signaling it's a container label one level deeper in the hierarchy. Other sibling L1 containers (e.g., other containers directly under a Global Account) use the same style too

---

## Service Box (White Rectangle, for Grouping)

> Related: official Areas → [the Areas section of `../GUIDELINES.md`](../GUIDELINES.md#areas-rectangular-zones). Match the corner radius to the official 16 px (`arcSize=32` + `absoluteArcSize=1`).

**⚠️ Usage rule**: services inside a Subaccount generally don't need to be enclosed in a rectangle (place just the icon + label). **Only enclose them in a rectangle when grouping a solution made up of multiple services, or when a service hosts other components (another service, an application, etc.).** Whether solution-level grouping is needed should be decided explicitly at the design stage.

**⚠️ Alternating fill rule (per the official Areas guidance)**: when nesting, alternate between fill (`#EBF8FF`) and white (`#ffffff`). If the parent is white, the child is filled; if the parent is filled, the child is white.

| Level | Example | fillColor |
|---|---|---|
| L0 | BTP container | `#EBF8FF` (filled) |
| L1 | Subaccount | `#ffffff` (white) |
| L2 | Service box (e.g. BWZ) | `#EBF8FF` (filled) |
| L3 | App container, etc. | `#ffffff` (white) |

```
style (L2, filled): rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#EBF8FF;
                     arcSize=32;absoluteArcSize=1;strokeWidth=1.5;
style (L3, white):  rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#ffffff;
                     arcSize=32;absoluteArcSize=1;strokeWidth=1;
```

### Service Box With a Service Icon

When a service is turned into a box because it hosts other elements, place **a service icon and title at the top-left of the box** to preserve the identity of that service. Leave the box's own `value` empty, and overlay an icon cell + title text cell as separate cells.

**Composition**:

1. **Service box** (`value=""`, using the service box style above)
2. **Service icon** (`shape=mxgraph.sap.icon;...;SAPIcon=<name>`, **32×32**)
   - Placed at `x+15, y+12` from the top-left of the box
   - No label needed, so `value=""`
3. **Title text** (`text;html=1;...;fontSize=12;fontStyle=1;fontColor=#002A86;fontFamily=Helvetica;`)
   - Placed to the right of the icon (roughly icon x + 40)
   - Align vertical center with the icon (same y as the icon, a text cell with height 32)
   - Uses the SAP BTP title color (`#002A86`) + bold
   - **Break service names of 3 or more words with `<br>`** (e.g., "SAP Build&lt;br&gt;Work Zone", "Cloud Foundry&lt;br&gt;runtime"). If you break the line, widen the text cell's height to 36–40

**Placement example** (for a BWZ box at `x=240, y=180, w=180, h=140`):

| Cell | x | y | w | h |
|---|---|---|---|---|
| Icon | 255 | 192 | 32 | 32 |
| Title | 295 | 192 | 120 | 32 |

**Child elements inside the box** (e.g., a UI5 App placeholder) are placed below the icon/title. Since the box's vertical center and the child elements' center will then differ, to keep straight horizontal arrows aligned with other services, **align the child elements' Y center with the main flow axis**, and size the box itself to contain that (leaving room above for the icon+title).

---

## Text

> Related: official Text → [the Text section of `../GUIDELINES.md`](../GUIDELINES.md#text). The official guidelines define a 4-tier text style hierarchy derived from Fiori Horizon, but the original text doesn't specify concrete font sizes — the `fontSize=12` used in this file is a project-specific operational value.

Text is used in two ways:

1. **As a standalone text cell** (`shape=text`)
2. **Embedded in a rectangle/icon's `value`, with the font specified via style** (recommended: for labels subordinate to a parent element, such as container names and service names)

### Medium Title

Used for subaccount names, service names, etc.

**Font properties (common):**

```
fontSize=12;fontColor=default;fontFamily=Helvetica;
```

#### When Used as a Standalone Cell

```
style: text;html=1;align=left;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;fontSize=12;fontColor=default;fontFamily=Helvetica;
```

#### When Embedded in a Container (e.g., a Subaccount Name)

Put the title into the container rectangle's `value`, and add the following to the style:

```
verticalAlign=top;align=left;spacingLeft=20;spacingTop=10;fontSize=12;fontColor=default;fontFamily=Helvetica;
```

- `verticalAlign=top;align=left` aligns it to the top-left
- Use `spacingLeft` / `spacingTop` to adjust the inner margin

#### When Embedded in an SAP Service Icon (a Service Name)

Just put the service name into the icon cell's (`shape=mxgraph.sap.icon`) `value`, and add the font properties to the style:

```
...;aspect=fixed;fontSize=12;fontColor=default;fontFamily=Helvetica;SAPIcon=<icon name>
```

- The icon base style's `verticalLabelPosition=bottom;align=center` centers the label below the icon

### Additional Information (Supplementary Text)

Text used for annotations and supplementary information.

```
style: text;html=1;align=left;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;fontSize=12;fontColor=#1D2D3E;fontFamily=Helvetica;
```

---

## Lines (Edges)

> Related: official Lines / Connectors → [the Lines / Connectors section of `../GUIDELINES.md`](../GUIDELINES.md#lines--connectors). The official guidelines distinguish usage by line type (solid = synchronous / dashed = asynchronous / dotted = optional / thick = firewall).

Lines connecting components. For how connection points are specified and when to use straight vs. bent lines, see [`layout.md`](layout.md).

> **Add a line jump (a gap at crossings) to every edge by default**: to show a "gap" where edges cross, **include `jumpStyle=gap;jumpSize=8;` in every line style**. Even in cases where crossings are unavoidable (e.g., the line from the left entry point to the central service crossing the line from the central service down to an external API), the line passing over gets a gap that keeps it readable. It's harmless to add it even to a line that never crosses another, so include it on every edge from the start. `jumpStyle` defaults to `gap` (a gap) (`arc`/`sharp` are also valid, but keep it consistent across the whole diagram).

### Straight Line

Used for components arranged in a straight line.

```
style: endArrow=blockThin;html=1;strokeColor=#475e75;bendable=1;rounded=0;endFill=1;endSize=4;strokeWidth=1.5;startSize=4;startArrow=none;startFill=0;jumpStyle=gap;jumpSize=8;
```

### Bent Line (Orthogonal)

Used for components that aren't arranged in a straight line.

```
style: edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;strokeColor=#475E75;fontFamily=Helvetica;fontColor=default;targetPerimeterSpacing=15;endSize=4;startSize=4;endArrow=blockThin;endFill=1;strokeWidth=1.5;startArrow=none;startFill=0;jumpStyle=gap;jumpSize=8;
```

### Bidirectional Arrow (trust / delegates auth)

Lines representing bidirectional relationships such as trust or authentication delegation. Color distinguishes the meaning.  Follows the official semantic colors (trust = pink / authentication = green / authorization = indigo):

| Relationship | Color | Example |
|---|---|---|
| trust | `#CC00DC` (pink) | Subaccount ↔ SAP Cloud Identity Services |
| delegates auth | `#188918` (green) | SAP Cloud Identity Services ↔ 3rd-party IdP |

In the official guidelines this green is assigned to **Authentication flows**; this project's "delegates auth" (authentication delegation) is positioned as one use of it.

When drawing arrowheads on both ends, add `startArrow=blockThin;startFill=1`. Match the label color to the line color too (e.g. `fontColor=#CC00DC`).

```
style: edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;
       strokeColor=#CC00DC;fontColor=#CC00DC;fontFamily=Helvetica;
       targetPerimeterSpacing=15;endSize=4;startSize=4;
       endArrow=blockThin;endFill=1;startArrow=blockThin;startFill=1;
       strokeWidth=1.5;fontSize=10;
text: trust
```

---

## SAP Service Icons (mxgraph.sap.icon)

> Related: the official Icons rule "SAP BTP Service Icons **must use the version with the grey background circle**" → [the Icons section of `../GUIDELINES.md`](../GUIDELINES.md#icons). The `mxgraph.sap.icon` shape satisfies this requirement in this format.

Using `shape=mxgraph.sap.icon` lets you specify an icon without embedding a base64 SVG. **This is the only format Claude handles for icons** (Generic Icons are the human's responsibility — see WORKFLOW.md).

### Common Base Style

```
shape=mxgraph.sap.icon;html=1;labelPosition=center;verticalLabelPosition=bottom;
align=center;verticalAlign=top;strokeWidth=1;strokeColor=#D5DADD;
fillColor=#EDEFF0;gradientColor=#FCFCFC;gradientDirection=west;aspect=fixed;
SAPIcon=<icon name>
```

Choose the size based on usage (also see the standard size table in [`layout.md`](layout.md)):

- **Standalone placement** (e.g., directly under a Subaccount): **50×50**
- **The identifying icon at the top-left of a service box**: **32×32** (see "Service Box With a Service Icon" above)

### Label Line-Break Rule (Determined by Placement Direction)

When an icon's `value` (the label text) is **3 words or more**, whether to break the line depends on the **placement direction**:

| Placement | Line break? | Reason |
|---|---|---|
| a. Elements arranged side by side | **Break the line** | Saves horizontal space |
| b. Elements arranged vertically (including standalone placement) | **Don't break the line** | Saves vertical space |

**When both a and b apply, a takes priority** (e.g., an icon inside a horizontal main flow gets treated as "side by side" and breaks its line, even if other elements are above/below it).

Line-break characters:
- Standalone SAP service icon (bottom label): `&#10;` (e.g., `value="SAP HANA&#10;Cloud"`)
- Title text at the top-left of a service box (separate cell): `<br>` (e.g., `value="SAP Build&lt;br&gt;Work Zone"`)

**Choosing where to break**:
- Treat brand/product names as a single unit (e.g., keep `SAP HANA` on one line, put `Cloud` on the next)
- Break at a point that balances the two lines (a 2:1 split is often natural)

> Before breaking a line, first consider whether the label text itself can be shortened. For the shortening policy, see the "Conventional Abbreviations for Service Names" section of `SKILL.md`'s Conventions.

### Label Position Variants (For the Exceptional Case of Placing It Above)

**The default bottom label is the rule** (`verticalLabelPosition=bottom;verticalAlign=top`). Work out the layout at the design stage so upward-pointing arrows don't happen in the first place (see SKILL.md Conventions §10).

Only when the layout truly forces an arrow into the bottom edge of the icon, **as an exception**, switch the label to the top:

| Label position | Style diff |
|---|---|
| Bottom (default) | `verticalLabelPosition=bottom;verticalAlign=top` |
| Top (exception) | `verticalLabelPosition=top;verticalAlign=bottom` |

When switching to a top label, ensure **at least 20 px of spacing** between it and the parent container's label (e.g., Subaccount) (too tight hurts readability).

> Design rule: [the "10. Avoid Upward-Pointing Arrows as a Rule" section of `../SKILL.md` Conventions](../SKILL.md#10-avoid-upward-pointing-arrows-as-a-rule-related-to-label-placement)

### Common SAPIcon Names

| Service | SAPIcon value |
|---|---|
| SAP Build Work Zone - Standard Edition | `SAP_Build_Work_Zone_-_Standard_Edition` |
| SAP Cloud Identity Services | `SAP_Cloud_Identity_Service` |
| SAP Task Center | `SAP_Task_Center` |
| SAP HANA Cloud | `SAP_HANA_Cloud` |
| SAP BTP, Cloud Foundry runtime | `SAP_BTP,_Cloud_Foundry_runtime` |
| SAP Build Process Automation | `SAP_Build_Process_Automation` |
| SAP AI Core | `SAP_AI_Core` |
| SAP AI Launchpad | `SAP_AI_Launchpad` |
| SAP Cloud Logging | `SAP_Cloud_Logging` |
| SAP Authorization and Trust Management service | `SAP_Authorization_and_Trust_Management_service` |
| SAP Integration Suite | `SAP_Integration_Suite` |
| SAP Integration Suite - Cloud Integration | `SAP_Integration_Suite_-_Cloud_Integration` |
| SAP Integration Suite - API Management | `SAP_Integration_Suite_-_API_Managment` |
| SAP Destination service | `SAP_Destination_service` |
| Cloud Connector | `Cloud_Connector2` |
| SAP Connectivity Service | `SAP_Connectivity_Service` |
| HTML5 Application Repository service | `HTML5_App_Repository` |
| SAP Custom Domain service | `SAP_Custom_Domain_service` |
| SAP Continuous Integration and Delivery | `SAP_Continuous_Integration_and_Delivery` |

Icon names for other services can be looked up in draw.io's shape library.

---

## Non-SAP Box (Grey Rectangle)

> Related: the official Foundation Primary Colors, "Non-SAP Area: Border `#475E75` / Fill `#F5F6F7`" → [the Colors section of `../GUIDELINES.md`](../GUIDELINES.md#colors)

A rectangle representing a non-SAP external service or system placed outside BTP (External LLM Provider, a 3rd-party API, etc.).

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#475E75;fillColor=#F5F6F7;
       arcSize=32;absoluteArcSize=1;strokeWidth=1.5;
       verticalAlign=middle;align=center;fontSize=12;fontColor=default;fontFamily=Helvetica;
text: Service name (e.g., External LLM Provider)
```

- Corner rounding matches the SAP boxes (`arcSize=32` + `absoluteArcSize=1`)
- The label is embedded in the box, centered
- Adjust the size to fit the content (guideline: 140×50)
