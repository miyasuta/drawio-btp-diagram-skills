# drawio-btp-diagram-skills

A Skill bundle for Claude Code / Claude Desktop. It provides a collaborative workflow that **turns a design you describe in words into an architecture diagram drawn in draw.io**.

---

## Overview / What It Does

When you draw an architecture diagram for SAP BTP, writing it directly in draw.io takes time, and correctly applying the official guidelines (colors, sizes, placement) is tedious. This skill handles it with a three-step process, ordered so each stage is easy to review:

1. You describe the design in words
2. Claude turns it into **logical structure in Mermaid** (`diagram.md`)
3. Once you agree on the structure, Claude generates the **draw.io XML** (`diagram.drawio`)
4. A **structural review** (checking the XML) and a **visual review** (checking a screenshot) run automatically

Reading draw.io XML directly is hard to review, so the key idea is to nail down the logic in Mermaid first, then translate it into XML.

### The Two Included Skills

| Skill | Role |
|---|---|
| `drawio-sap-diagrams` | The 5-step collaborative workflow (words → Mermaid → XML) |
| `sap-diagram-review` | Automatic review of the generated XML / screenshots |

`sap-diagram-review` has its own independent triggers (XML generation complete / screenshot shared), so it can also be called on its own from other workflows.

### Who This Is For

- Architects and developers building apps or solutions on SAP BTP
- Anyone who wants diagrams that follow the official SAP visual guidelines

---

## Getting Started

### Prerequisites

- Claude Code installed
- `git` / `ln` available (macOS / Linux)

### 1. Fork This Repository

This repo is meant to be used as your own copy, not pulled directly from the original — as covered in [How the Skill Grows](#how-the-skill-grows) below, Claude will keep editing and committing to it as you use it. Fork it on GitHub first, then clone your fork.

### 2. Clone + Symlink

Clone to any location you like — `<path-to-clone>` below is just a placeholder:

```bash
git clone https://github.com/<your-fork>/drawio-btp-diagram-skills.git <path-to-clone>
ln -s <path-to-clone>/skills/drawio-sap-diagrams ~/.claude/skills/drawio-sap-diagrams
ln -s <path-to-clone>/skills/sap-diagram-review     ~/.claude/skills/sap-diagram-review
```

Using symlinks means `git pull` results take effect immediately (`cp -r` is not recommended, since it's easy to forget to update the copy).

To verify:

```bash
ls -la ~/.claude/skills/drawio-sap-diagrams/        # confirm the symlink resolves
ls ~/.claude/skills/drawio-sap-diagrams/references/ # layout / elements / guidelines
```

### 3. Restart Claude Code

If `drawio-sap-diagrams` and `sap-diagram-review` both appear in the skill list at the start of a session, you're ready to go.

### 4. Draw Your First Diagram

Try it out in any project:

```
Please draw an architecture diagram for this project (CAP + UI5).
- The UI should be accessed from Work Zone
- CAP connects to S/4HANA (Private Cloud)
```

Claude will ask numbered follow-up questions for any missing information; answer by number and it will generate a Mermaid diagram, then the draw.io XML. Paste the XML into draw.io, attach a screenshot to the conversation, and the visual review will run automatically too.

See [Usage](#usage) below for the detailed flow and more example prompts.

---

## Usage

### Sample Prompt (Recommended: Start From an Ambiguous State)

If the project already has an implementation, Claude can read the code to work out the fine-grained structure, so you only need to give it **the high-level relationships**:

```
Please draw an architecture diagram for this project (CAP + UI5).
- The UI should be accessed from Work Zone
- CAP connects to S/4HANA (Private Cloud)
```

If information is missing, Claude will **ask numbered follow-up questions**. Answer them briefly by number, and once you agree on the structure it moves on to Mermaid → element inventory → draw.io XML generation.

```
1. Yes / 2. Via CIS, no 3rd-party / 3. Yes / 4. Not needed
```

What Claude asks depends on your request and how well it already understands the project (the example above is just meant to illustrate the reply style).


### Workflow Overview

Once you send a prompt, the skill proceeds through the following steps:

1. **Generate Mermaid** → outputs the logical structure to `diagram.md`. Review it in the Mermaid preview and give correction instructions in words if needed
2. **Element inventory** (a text list of cells without coordinates) → fix any structural mistakes here
3. **Generate draw.io XML** → `diagram.drawio`
4. **Structural review** (`sap-diagram-review` runs automatically)
5. Paste it into draw.io and attach a screenshot to the conversation
6. **Visual review** (`sap-diagram-review` runs automatically)

---

## How the Skill Grows

This skill is designed to **grow as you use it**. When you notice an element that isn't yet in the shipped `references/`, or a preference or house rule of your own, just mention it in conversation and Claude will persist it to the right file.

This all happens inside **your own fork** — it never touches the original repository you forked from. Whether to upstream any of it via a pull request is entirely up to you.

### Examples of How It Grows

| Trigger | Where it gets persisted |
|---|---|
| A new style, color, or size (documented officially but not yet defined in the skill) | `references/elements.md` or `references/layout.md` |
| A layout policy like "this pattern should generally be arranged like this" | The Conventions section of `SKILL.md` |
| A design rule like "these services are always drawn together" | The Conventions section of `SKILL.md` |
| A personal diagram preference, such as "avoid upward-pointing arrows" | The Conventions section of `SKILL.md` |

When Claude itself judges that something "looks reusable," it will suggest adding it, and it gets written once you approve. You can also instruct it explicitly:

```
I want to keep using this layout policy going forward, so please add it to the Conventions section of SKILL.md.
```

### Changes Take Effect Immediately via Symlink

Because the skill is installed via a symlink, when Claude edits `references/` or `SKILL.md`, **it directly rewrites the files in the repository itself**. The new rules take effect starting with the next session.

### Auto-Commit (Optional)

Because the skill repo is symlinked, updates made during **a session in the consuming project** are not `git commit`ed by that session. Left unchecked, uncommitted changes will pile up.

If you'd rather not commit these changes by hand, you can optionally set up a Stop hook in `~/.claude/settings.json` to automatically commit any uncommitted changes in the skill repo **when a session ends**. Replace `SKILL_REPO`'s value below with wherever you actually cloned this repo:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "SKILL_REPO=<path-to-clone>; if [ -d \"$SKILL_REPO/.git\" ] && [ -n \"$(git -C \"$SKILL_REPO\" status --porcelain 2>/dev/null)\" ]; then git -C \"$SKILL_REPO\" add -A && FILES=$(git -C \"$SKILL_REPO\" diff --cached --name-only | paste -sd , - | sed 's/,/, /g') && STAT=$(git -C \"$SKILL_REPO\" diff --cached --shortstat | sed 's/^ *//') && git -C \"$SKILL_REPO\" commit -m \"Auto-update $FILES ($STAT)\"; fi || true"
          }
        ]
      }
    ]
  }
}
```

Behavior:

- If a session has no uncommitted changes in the skill repo → does nothing
- If a session has changes → commits with a message that includes diff info, like `Auto-update skills/drawio-sap-diagrams/references/elements.md (1 file changed, 1 insertion(+))`
- Never touches the repository of the consuming project (scope is fixed via `git -C "$SKILL_REPO"`)

After adding the config, restart Claude Code, or open `/hooks` once, for it to take effect.

---

## Per-Project Customization

Project-specific output path rules belong in **each project's `CLAUDE.md`**. The skill itself does not hold any project-specific paths.

### Default

If `CLAUDE.md` doesn't specify an output path, the skill suggests the following as the default:

```
diagrams/diagram.md       # Mermaid (logical structure)
diagrams/diagram.drawio   # draw.io XML (for pasting)
```

---

## Repository Structure

```
drawio-btp-diagram-skills/
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    ├── drawio-sap-diagrams/
    │   ├── SKILL.md          # 5-step workflow + Conventions (inline)
    │   └── references/
    │       ├── layout.md     # draw.io implementation details (px sizes, coordinates, edge connection points)
    │       ├── elements.md   # draw.io element style definitions (BTP, logos, icons, lines)
    │       └── guidelines.md # English summary of SAP's official guidelines (source of truth)
    └── sap-diagram-review/
        └── SKILL.md          # The two phases: structural review + visual review
```

---

## License

[MIT](LICENSE)
