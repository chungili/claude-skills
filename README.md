# Claude Skills

A collection of custom slash commands for [Claude Code](https://claude.ai/code).

---

## Available Skills

### `/polish-inline` — Academic English Editor

Polishes an academic document **in place** using Git-style inline diff markings, so you can review every change before accepting it.

#### What it does

- Rewrites weak or informal phrasing using academic alternatives
- Marks every change inline:
  - `~~original text~~` — content to be removed
  - **revised text** — the replacement
- Appends a `## Rationale` table at the end of the file explaining each change

#### Installation

Copy the command file into your project's `.claude/commands/` directory:

```bash
# Clone this repo
git clone https://github.com/chungili/claude-skills.git

# Copy the skill to your project
cp claude-skills/.claude/commands/polish-inline.md your-project/.claude/commands/
```

Or install it globally for all projects:

```bash
cp claude-skills/.claude/commands/polish-inline.md ~/.claude/commands/
```

#### Usage

Inside Claude Code, run:

```
/polish-inline <filename>
```

**Example:**

```
/polish-inline paper.qmd
```

Claude will read the file, apply inline edits, and append a rationale table like this:

| # | Original | Revised | Reason |
|---|----------|---------|--------|
| 1 | The model gets a good result. | The model yields satisfactory performance. | Replace vague verb "gets" with precise academic alternative |
| 2 | A lot of studies show that... | Numerous studies demonstrate that... | Prohibited pattern removed; vague verb replaced |

#### Academic Standards Applied

| Rule | Example |
|------|---------|
| No contractions | `don't` → `do not` |
| Precise verbs | `get` → `obtain`, `show` → `demonstrate` |
| No redundant phrases | ~~"it is important to note that"~~ |
| Formal connectives | `but` → `nevertheless`, `also` → `furthermore` |
| No prohibited patterns | ~~"A lot of"~~, ~~"Game changer"~~, ~~"In the modern era"~~ |

---

### `/qmd2revealjs` — Quarto to RevealJS Slide Converter

Converts a Quarto `.qmd` document (lecture notes, analysis reports, tutorials) into a polished RevealJS slide deck, preserving every word of content.

#### What it does

- Restructures the document into slides following the rule **one concept per slide**
- Replaces the YAML header with a RevealJS-compatible configuration
- Handles overflow automatically (prefers `{.smaller}` and slide splits over scroll bars)
- Produces two output files: `<name>_slides.qmd` and a companion `custom.css`
- Appends an overflow handling summary as an HTML comment at the end

#### Installation

```bash
# Clone this repo
git clone https://github.com/chungili/claude-skills.git

# Copy the skill to your project
cp claude-skills/.claude/commands/qmd2revealjs.md your-project/.claude/commands/

# Copy the companion CSS (required for rendering)
cp claude-skills/custom.css your-project/
```

Or install the skill globally for all projects:

```bash
cp claude-skills/.claude/commands/qmd2revealjs.md ~/.claude/commands/
```

> `custom.css` must reside in the **same directory as your `.qmd` file** for Quarto to pick it up during rendering. The skill references it via `css: custom.css` in the YAML header.

#### Usage

Inside Claude Code, run:

```
/qmd2revealjs <filename>
```

**Example:**

```
/qmd2revealjs MultipleReg.qmd
```

Claude will produce `MultipleReg_slides.qmd` and `custom.css` in the same directory.

> **Note:** A ready-to-use `custom.css` is included in this repo. Copy it to your project directory before rendering:
> ```bash
> cp claude-skills/custom.css your-project/
> ```
> The file defines the slide theme (accent colour `#ffd54a`, code block sizing, callout styles, etc.). `/qmd2revealjs` will preserve all existing classes if the file is already present, or generate a fresh one if it is not.

#### Key Conversion Rules

| Rule | Detail |
|------|--------|
| Heading mapping | `#` → section title slide, `##` → one slide, `###` → sub-heading within slide |
| Overflow priority | `{.smaller}` → `.columns` layout → slide split → `{.scrollable}` (last resort) |
| Math derivations | Every step preserved; 4+ steps split across slides, final result in callout |
| Code chunks | All chunk options (`label`, `fig-cap`, `echo`) kept intact |
| Omitted content | Administrative content (edit logs, rationale tables) excluded from slides |

---

## Adding More Skills

Place any `.md` file in `.claude/commands/` and it becomes a slash command automatically. The filename (without `.md`) is the command name.
