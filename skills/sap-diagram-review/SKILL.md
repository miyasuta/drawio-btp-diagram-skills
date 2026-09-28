---
name: sap-diagram-review
description: Use after generating or editing a draw.io diagram of SAP BTP architecture, or when the user shares a diagram screenshot — checks XML structure (Phase 1) and rendered visuals (Phase 2) against SAP visual conventions and the drawio-sap-diagrams skill's references.
---

# SAP Diagram Review

## Overview

Reviews draw.io XML in **two phases**.

1. **Structural review (Step 4.5)**: runs automatically right after XML generation. Checks structure and style against the documentation
2. **Visual review (Step 5.5)**: after the human pastes it into draw.io, checks the appearance via a screenshot

## When to Use

- **Step 4.5**: runs automatically right after XML generation completes
- **Step 5.5**: runs automatically when the human provides a screenshot
- Can also run when the human explicitly says "review this"

### Cases Requiring Re-Review (MUST)

After making any of the following changes to an existing `diagram.drawio`, **you must re-run the structural review (Phase 1)**:

- **Adding or removing** an element (icon, box, edge, etc.)
- **Moving** an element (across containers, or a significant change in position — e.g., top → bottom)
- **Resizing** a container (width / height)
- Changing the main flow's **center axis** (Y or X)

**Reason**: when an element's position or size changes, the results of dependent checks — container margins, sizing consistency across sibling boxes, coordinate alignment — change along with it. Even if the diff looks small, the related margin calculations can break across the whole diagram, so a full re-evaluation across all categories is needed.

**Past failure example**: when HTML5 App Repo / XSUAA were moved from the top row of a container to the bottom row, the main flow's Y coordinate was left unchanged, leaving over 100 px of empty space at the top of the Subaccount. Running the re-review at that point would have caught it immediately via the `layout.md §2` check, "avoid wasted space" / "check for unneeded margins in both directions."

---

## Phase 1: Structural Review (Step 4.5)

### Process

1. Read the target `diagram.drawio`
2. Read the corresponding `diagram.md` (Mermaid)
3. Read the following documents to get the current rules:
   - The Conventions section of the `drawio-sap-diagrams` skill
   - `../drawio-sap-diagrams/references/layout.md`
   - `../drawio-sap-diagrams/references/elements.md`
4. Check each category and collect any problems
5. Report a list of problems if any are found; otherwise report "Review OK"

### Review Categories

Each category is judged by referencing the documentation. Rather than a fixed checklist, **the current content of the documentation** is the rule.

| Category | Reference | What to check |
|---|---|---|
| Composition patterns | `drawio-sap-diagrams` Conventions | End User+AC pairing, CIS placement, the on-premise connection 3-piece set, consolidating arrows from a service box |
| Labels / language | `drawio-sap-diagrams` Conventions | Whether element names are in English, whether arrow labels are on the allowed list |
| Container fill | `../drawio-sap-diagrams/references/elements.md` | Whether the L0-L3 alternating rule (#EBF8FF / #ffffff) is followed |
| Icon format | `../drawio-sap-diagrams/references/elements.md` + `layout.md` | SAP icon size, presence of `html=1;`, whether Generic Icons use the placeholder format (including applying known aspect ratios), whether the label is a separate cell |
| Label line breaks | `../drawio-sap-diagrams/references/elements.md` | Whether labels of 3 or more words are broken (SAP service icon: `&#10;` / service box title: `<br>`). Not breaking wastes horizontal space |
| Label position | `../drawio-sap-diagrams/references/elements.md` (Label Position Variants) + `SKILL.md §10` | Whether SAP service icon labels are consistently at the **default bottom** (`verticalLabelPosition=bottom`). If any icon is switched to `top`, verify whether it **truly** meets the exception conditions in §10 (unable to avoid entering the bottom edge, etc.). If it doesn't meet them, or there's an alternative (keep the bottom label and route the arrow out below the label, or move the source), **flag it 🚩** — don't wave it through as "an intentional deviation, no fix needed." As a rule, flag any diagram where label positions are mixed top/bottom |
| Edge approach | `../drawio-sap-diagrams/references/layout.md` §1b | Whether there's any "L-shaped approach" into a box (entering a non-facing edge, such as coming from the side and piercing the bottom/top edge). If so, suggest straightening it (adjusting the source position, or multiple straight lines into different points on the same edge) |
| Line jump | `../drawio-sap-diagrams/references/elements.md` (Lines/Edges) | Whether every edge's style includes `jumpStyle=gap;jumpSize=8;` (a gap at crossings). Suggest adding it to any edge missing it |
| Container size | `../drawio-sap-diagrams/references/layout.md` | Top margin for a labeled container (text label: 50–60px), bottom/side margins including SAP icon label overflow, centering when there's a single child element, unifying **size and alignment** across sibling boxes at the same level (height + top Y for horizontal arrangement, width + left X for vertical; check by coordinates whether only the right/bottom edge matches while the opposite edge is misaligned) |
| Container margins (left/right symmetry) | `../drawio-sap-diagrams/references/layout.md` §2 | **Verify by coordinates that the left/right margins of nested containers (e.g., Subaccount within BTP) are equal.** Compute `inner.x − outer.x` (left margin) and `outer.right − inner.right` (right margin); flag 🚩 if they differ. Also check each margin falls within 30–55px. Top/bottom asymmetry is acceptable when another container (e.g. CIS) hangs below, but **left/right must always be symmetric**. Redo this check whenever elements move or a container's width changes |
| Title overflow | `../drawio-sap-diagrams/references/elements.md` + `layout.md` | Whether a service box's title text cell is wide enough for the title string (estimate roughly 8px per character for bold fontSize=12) |
| Coordinate alignment | `../drawio-sap-diagrams/references/layout.md` | Whether the center coordinates of elements connected by a straight arrow match |
| Edge connection points | `../drawio-sap-diagrams/references/layout.md` | Whether entryX/Y, exitX/Y are explicit on important edges such as trust / authenticate |
| Required elements | Cross-check against `diagram.md` | Whether every node and edge defined in Mermaid exists in the XML |

### Output Format

If there are problems:

```
## Diagram Review (Structural)

- [ ] **Composition pattern**: Cloud Connector is placed outside the S/4HANA box (Conventions §8)
- [ ] **Coordinate alignment**: SPA and DEST centers are off by 2px on Y (layout.md §4)

Should I fix the above?
```

If there are no problems:

```
## Diagram Review (Structural)

Review OK — please paste it into draw.io and share a screenshot
```

---

## Phase 2: Visual Review (Step 5.5)

### Trigger

Triggered when the human **attaches a screenshot to the conversation** after pasting the XML into draw.io. Claude reads the image with Read and checks it from the following angles.

### Checkpoints

| Category | What to check |
|---|---|
| Label overflow | Whether an SAP service icon's label overflows the container border |
| Title overflow | Whether a service box's title (the text next to the icon) overflows the box border |
| Label overlap | Whether adjacent icons' labels overlap each other. Whether a downward arrow and the label it originates from are touching too closely (is there a gap between the label text and the line? → `layout.md §6`) |
| Consistent label position | Whether icon labels are consistently on the bottom (is there a mix of some on top?). If mixed, check whether §10's exception conditions apply; if not, suggest unifying them to the bottom |
| Edges vs. container borders | Whether an edge crossing a container boundary has its **bend point or label overlapping the container's border line** (`layout.md §1c`). If so, suggest moving the external target further away, or moving the bend point into a clear zone outside the border via a waypoint |
| Edge label placement | Whether an edge label sits **on top of a bend point (corner)**, hiding the line's shape, or **floats away from the line**, making it unclear which edge it belongs to (`layout.md §1d`). Suggest placing it on a straight segment, centered on the line, and aligning the height (Y) of parallel labels |
| Consistent box size/alignment | Whether sibling service boxes at the same level have matching height + top edge (horizontal arrangement) or width + left edge (vertical arrangement) — i.e., whether they look uneven even when sizes match, due to edge misalignment |
| Container margins | Whether the container has appropriate margins on all sides (too tight / too loose). **Whether nested containers' left/right margins look equal** (if one side is wider, the inner container looks off-center). Suggest symmetrizing if left/right are asymmetric |
| Arrow straightness | Whether an arrow that should be straight is bent. Whether there's an L-shaped approach into a box (entering from the side and piercing the bottom/top edge) |
| Element visibility | Whether text is a readable size, whether icons look squashed |

### Process

1. Review the screenshot image
2. Identify any problems using the checkpoints above
3. Report specific issues and suggested fixes if any are found; otherwise report "Visual review OK"

### Output Format

If there are problems:

```
## Diagram Review (Visual)

- [ ] **Label overflow**: the Cloud Connector label overflows past the bottom of the S4 box → extend the height of S4
- [ ] **Arrow straightness**: the trust arrow is bent → the center X of Sub and CIS don't match

Should I fix these?
```

If there are no problems:

```
## Diagram Review (Visual)

Visual review OK
```
