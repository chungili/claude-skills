---
name: qmd2revealjs
description: >
  Convert any Quarto (.qmd) document — lecture notes, analysis reports, or
  tutorials — into a polished RevealJS slide deck. Use this skill whenever
  the user asks to "turn this qmd into slides", "convert to RevealJS",
  "make a slide deck from this document", or uploads a .qmd file and wants
  a presentation. Also triggers when the user says "換成投影片", "轉成 revealjs",
  or any similar phrase in any language. Always produces two output files:
  the converted _slides.qmd and a companion custom.css.
---

# QMD → RevealJS Slide Converter

Converts a Quarto `.qmd` document into a RevealJS slide deck while preserving
every word of content. The only changes are structural: YAML header, slide
pagination, and overflow handling.

---

## Core Principles

1. **Content is sacred** — every sentence, equation, code block, and figure
   caption is preserved verbatim. Nothing is summarised or dropped.
2. **One concept per slide** — split by idea, not by line count.
3. **Prefer more slides over crowded slides** — when in doubt, split.
4. **No scroll bars** — if content overflows, fix it with `{.smaller}` or
   split the slide; never let a slide silently scroll.
5. **`{.smaller}` before `{.scrollable}`** — shrink font before adding a
   scroll bar. Use `{.scrollable}` only as a true last resort.

---

## Step 1 — Read Source Files First

Before writing a single line, read:

```
view /mnt/project/<name>_slides.qmd   # past conversion examples (style reference)
view /mnt/project/custom.css          # existing CSS — preserve every class
```

Extract from these references:
- **Pagination granularity** (how much content per slide)
- **`.columns` double-column patterns**
- **Callout usage habits**
- **Summary table formats at section ends**

---

## Step 2 — YAML Header

Replace the entire `format:` block with:

```yaml
format:
  revealjs:
    slide-level: 2
    theme: default
    transition: slide
    scrollable: true
    smaller: true
    code-fold: false
    code-overflow: scroll
    number-sections: false
    toc: true
    toc-depth: 1
    toc-title: "Contents"
    slide-number: true
    chalkboard: true
    footer: "（課程標題）"
    css: custom.css
```

**Keep unchanged:**
```yaml
execute:
  cache: true        # or false — match source
  warning: false
  fig-align: center
  out-width: "XX%"
  echo: true         # or false — match source
```

**Remove these HTML/PDF-only keys** (they cause render errors in RevealJS):
`grid`, `css` (original), `reference-location`, `citation-location`,
`documentclass`, `include-in-header`, `geometry`, `number-depth`,
`embed-resources`, `code-tools`, `theme: cosmo` (html themes).

---

## Step 3 — Heading Level Mapping

`slide-level: 2` means:

| Original | RevealJS role |
|----------|---------------|
| `#`      | Section title slide (TOC entry) |
| `##`     | One slide (pagination unit) |
| `###`    | Sub-heading **within** a slide — does NOT split |
| `---`    | Force split with no title (for continuous derivations) |

**Upgrade rule:** if a `###` should become its own slide, promote it to `##`.
Previously `##` sections may need to become `#` chapter markers.

---

## Step 4 — Overflow Decision Tree

When a slide has too much content, work through this priority list **in order**:

```
Content fits comfortably?
  └── Yes → leave as is

Slightly over (< 20% extra)?
  └── {.smaller} on that slide only:  ## Title {.smaller}

Structured content (formula + explanation, two figures)?
  └── ::: {.columns} double-column layout

Clearly too much for one slide?
  └── Split with --- (no title, keeps flow) or new ## Title (cont.)

Code chunk + its output both long?
  └── Split into "Code" slide + "Output / Discussion" slide

Multiple callouts stacked?
  └── Each callout gets its own slide, or pair them with .columns

Long mathematical derivation (4+ steps)?
  └── 2–3 steps per slide; label as (1), (2), (3)
      Last page of derivation: wrap final result in callout-note

Nothing works?
  └── {.scrollable} as absolute last resort
```

### Quick-Reference Table

| Problem | Tool | Syntax |
|---------|------|--------|
| Slightly over budget | Shrink font | `## Title {.smaller}` |
| Derivation too long | Split pages | `---` between steps |
| Derivation needs TOC | Named split | `## Deriving X (cont.)` |
| Short steps, same slide | Reveal one at a time | `::: {.fragment}` |
| Figure + explanation | Side by side | `::: {.columns}` |
| Many bullet items | Progressive reveal | `::: {.incremental}` |
| Dense derivation | Extreme shrink | `## Title {.tiny}` |
| Truly unsplittable | Last resort | `## Title {.scrollable}` |

---

## Step 5 — Content-Type Handling

### Mathematical derivations
- Every intermediate step is kept — never condense algebra.
- 4+ steps → split into multiple slides, label `(1)`, `(2)`, `(3)`.
- Final result of each proof → wrap in `::: {.callout-note}`.
- If a derivation page still overflows after splitting → add `{.smaller}`.

### Code chunks
- Keep all `#| label:`, `#| fig-cap:`, `#| echo:` options intact.
- If a chunk + its R output together overflow → split: code slide first,
  then output/discussion slide.
- Line highlighting (`#| code-line-numbers:`) is preserved as-is.

### Figures (static PNG/SVG)
- Keep `fig-align`, `out-width`, `fig-cap` attributes unchanged.
- If a figure is very tall, reduce `out-width` slightly (e.g. `"55%"` → `"48%"`).
- Plotly/HTML widgets: keep explicit `width=` and `height=` set in `plot_ly()`.

### Tables
- Markdown tables stay as-is; long data tables get `{.smaller}`.
- If a table is the *only* content, it can be slightly wider (`out-width: "80%"`).

### Callout boxes
- Multiple callouts stacked on one page → split or use `.columns`.
- Keep `## Title` inside callout when present.
- Callout type stays the same (note / warning / tip / important).

### Non-teaching content
- Appendices, edit logs, rationale tables, bibliography notes that are
  administrative rather than pedagogical → **omit from slides**.

---

## Step 6 — Output Files

Always produce exactly two files:

### File 1: `<original_name>_slides.qmd`
- Complete slide deck — never truncate.
- YAML header per Step 2.
- `css: custom.css` in the YAML.
- After the last slide, append a **"Overflow Handling Summary"** table:

```markdown
<!-- Overflow Handling Summary
| Slide title | Problem | Solution |
|-------------|---------|----------|
| ...         | ...     | ...      |
-->
```
(Embed as HTML comment so it doesn't render but is readable in source.)

### File 2: `custom.css`
Build from the project's existing `custom.css` (Step 1) and add/adjust:

| Section | What to include |
|---------|----------------|
| `.my-col` | Gold accent `#ffd54a`, bold — **always preserve** |
| `pre` / `pre code` | `font-size: 0.62em`, tight `line-height` |
| Cell output | `font-size: 0.58em` — tighter than input |
| `h1/h2/h3` | Compact `margin-bottom`, `line-height: 1.15` |
| `h2` | `font-size: 1.5em` |
| Math display | `margin: 0.4em 0 !important` |
| `.callout` | Compact padding |
| `.callout-note` | Yellow `#ffd54a` border + body text; white header bg |
| Callout icon | Yellow filter for note icon |
| Math in callout | `color: #ffd54a !important` for KaTeX/MathJax |
| `table` | `font-size: 0.85em`, compact cell padding |
| `ul/ol/li` | Tight margins |
| `p` | `margin: 0.35em`, `line-height: 1.35` |
| `.columns` | `font-size: 0.95em` |
| Static images | `max-height: 60vh` |
| Plotly widgets | `max-height: 92vh`, `max-width: 100%` |
| `figcaption` | `font-size: 0.75em` |
| `section.tiny` | `font-size: 0.55em`, keep `h2` at `1.5em` |
| `.slide-number` | `font-size: 0.75em`, muted colour |

> If the project has no existing `custom.css`, generate a fresh one from
> the table above. If it exists, **preserve every class** and only add/edit.

---

## Step 7 — Final Checklist

Before outputting files, verify:

- [ ] Every section, paragraph, equation, and code chunk from the source is present
- [ ] No slide has `{.scrollable}` unless it was truly unavoidable
- [ ] `{.smaller}` slides are listed in the overflow summary
- [ ] Code chunk options (`label`, `fig-cap`, `echo`, `out-width`) are intact
- [ ] `bibliography: ref.bib` is in YAML if the source had it
- [ ] `custom.css` retains `.my-col` and all pre-existing classes
- [ ] Both files are written to `/mnt/user-data/outputs/` and presented with `present_files`

---

## Common Mistakes to Avoid

| Mistake | Correct approach |
|---------|-----------------|
| Removing intermediate algebra steps "for brevity" | Keep every step — math is the teaching content |
| Putting `{.scrollable}` on overflowing slides by default | Use `{.smaller}` or split first |
| Promoting every `###` to `##` (creating too many slides) | Only promote `###` when content warrants its own slide |
| Merging multiple short code chunks into one slide | Keep one operation per slide |
| Keeping editorial/administrative content (edit logs, rationale tables) | Omit non-teaching content |
| Forgetting `echo: true` for slides meant to teach code | Match source `echo` setting |
| Writing overflow summary inline (renders as slide content) | Use HTML comment `<!-- ... -->` |
