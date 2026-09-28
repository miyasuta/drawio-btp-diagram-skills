# SAP Official BTP Solution Diagram Guidelines (Summary)

This document summarizes the key rules from SAP's official guidelines, [SAP/btp-solution-diagrams](https://github.com/SAP/btp-solution-diagrams), in English for convenient offline reference.

## Where This Document Fits

- **The official SAP guidelines (this file) = source of truth**
- **[`styles/`](styles/) = the implementation layer that realizes the official guidelines as draw.io XML**
- When `styles/` and the official guidelines disagree, `styles/` should generally be fixed to match the official guidelines (confirm with the user before changing anything)

The original text is in English, and many of the numeric specs live in images (PNG) or in the draw.io library rather than in prose. This file only captures the rules that can be read as text; for details (the 4-tier Text sizes, the concrete Numbers style, exact Icon sizes, etc.) refer to the original images and the draw.io library.

Original repository: https://github.com/SAP/btp-solution-diagrams

---

## Foundation

Original: [`guideline/docs/btp_guideline/foundation.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/foundation.md)

### Colors

Horizon is SAP's default product visual style. "Its color balance helps to draw the user's attention to the essential information and functions."

#### Primary Colors

| Usage | Border | Fill |
|---|---|---|
| **SAP / BTP Area** | `#0070F2` | `#EBF8FF` |
| **Non-SAP Area** | `#475E75` | `#F5F6F7` |

| Usage | Color |
|---|---|
| **Text — Title** | `#1D2D3E` |
| **Text — Body** | `#556B82` |

#### Semantic Colors

| Meaning | Border | Fill |
|---|---|---|
| **Positive** | `#188918` | `#F5FAE5` |
| **Critical** | `#C35500` | `#FFF8D6` |
| **Negative** | `#D20A0A` | `#FFEAF4` |

#### Accent / Emphasized Colors

| Name | Border | Fill |
|---|---|---|
| **Teal** | `#07838F` | `#DAFDF5` |
| **Indigo** | `#5D36FF` | `#F1ECFF` |
| **Pink** | `#CC00DC` | `#FFF0FA` |

### Line Styles

Original: *"a solid arrow means direct communication, while a dashed arrow implies indirect communication."*

Recommended line styles for BTP Solution Diagrams:

| Line style | Usage |
|---|---|
| **Solid** | direct, synchronous request-response data flows |
| **Dashed** | indirect, asynchronous data flows |
| **Dotted** | optional data flows |
| **Thick** | firewalls only |

### Spacing

Original: *"Spacing around objects should be even and roughly the height of the SAP Logo."*

Keep spacing between objects even, roughly the height of the SAP logo. See the image (spacing.png) for concrete numbers.

---

## Diagram Components

Original directory: [`guideline/docs/btp_guideline/diagr_comp/`](https://github.com/SAP/btp-solution-diagrams/tree/main/guideline/docs/btp_guideline/diagr_comp)

### Areas (Rectangular Zones)

Original: [`diagr_comp/areas.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/areas.md)

- **Corner radius**: *"A fixed corner radius of 16 pixels is recommended"* — a **fixed 16 px**
- **Colors**: follow the Foundation Primary Colors (SAP = blue, non-SAP = grey)
- **Use of accent colors**: don't overuse them, or they'll overwhelm the diagram
- **Nesting**: *"alternate between using a fill and not using a fill"* — when nesting areas, alternate whether they're filled, to keep contrast
- **Cardinality**: group same-kind elements with a stacked representation, keeping style consistent
- **Hierarchy**: the parent layer uses the BTP (blue) style; child elements alternate fill patterns

### Icons

Original: [`diagr_comp/icons.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/icons.md)

There are two icon sets.

#### SAP BTP Service Icons

- Icons for every service listed in the SAP Discovery Center
- **Mandatory rule**: *"For diagram visualization it is mandatory to use the version with grey background circle."*
- A draw.io library is provided in the GitHub repository

#### Generic Icons

- For generic elements like devices and databases, or content with no dedicated visual representation
- *"The primary color used is neutral grey."*
- An icon set with a soft gradient
- A draw.io library is provided in the GitHub repository

### Lines / Connectors

Original: [`diagr_comp/lines_connectors.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/lines_connectors.md)

Definition: *"Connectors are non-, one- or bidirectional arrows."*

#### Direction (Data Flow)

| Type | Meaning |
|---|---|
| **One-directional** | *"typically represent a request-response from client to server"* |
| **Bidirectional solid arrows** | *"may represent mutual trust"* |

#### Semantic Color Mapping

Use standardized semantic colors for recurring flow types:

| Flow type | Color |
|---|---|
| **Trust flows** | Pink (`#CC00DC`) |
| **Authentication flows** | Green (`#188918`) |
| **Authorization flows** | Indigo (`#5D36FF`) |

Use accent colors to emphasize non-semantic paths or flows.

#### Firewalls / Network Boundaries

Represented with thick grey lines.

#### Interfaces

A dedicated visual representation is provided in the draw.io library (see the interfaces image in the original for details).

### Numbers

Original: [`diagr_comp/numbers.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/numbers.md)

- *"Numbers can be used to describe specific paths in diagrams."*
- For the concrete style (size, color, font), see the original image (numbers.png) and the draw.io library
- The original text has no numeric spec

### Product Names (Product Name Labeling)

Original: [`diagr_comp/product_names.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/product_names.md)

- *"SAP product names must be paired with the SAP logo"*
- *"It is not recommended to use too many SAP logos in the same diagram"*
- To avoid overusing the logo, use a **text-only element** as an alternative

### Text

Original: [`diagr_comp/text.md`](https://github.com/SAP/btp-solution-diagrams/blob/main/guideline/docs/btp_guideline/diagr_comp/text.md)

- *"To create hierarchies four text styles were derived from Fiori Horizon."*
- *"Texts defined in this document may need to be scaled up to fit the specifications of the target medium."*
- The concrete font size, color, and typeface for the 4 tiers are in the images (text_styles.png, text_element_hierachy.png) and the PowerPoint / draw.io library
- The original text has no numeric spec

---

## Tips for Consulting the Official Source

- When you need concrete numbers (number styles, the Text hierarchy, icon px sizes, etc.), first check the images on the original page, then check the draw.io library XML under [`assets/shape-libraries-and-editable-presets/draw.io`](https://github.com/SAP/btp-solution-diagrams/tree/main/assets/shape-libraries-and-editable-presets/draw.io) in the GitHub repository
- This file only captures rules that could be read from the original text. Rules documented only in images should be checked against the original separately
- If you find a discrepancy between the official rules and the `styles/` implementation, don't fix it unilaterally — report it to the user first, then act
