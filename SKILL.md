---
name: template-constrained-ppt-generation
description: "Use when generating an enterprise PowerPoint from an approved template, or when reverse-engineering approved reference decks into a reusable constrained PPT template. Separates content decisions from visual decisions, permits only registered components and placeholders, handles overflow through compression and approved page duplication, and rejects decks that fail structural or visual validation."
version: 1.0.0
author: Tricro968
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ppt, powerpoint, templates, enterprise, visual-constraints, validation, document-generation]
    related_skills: []
---

# Template-Constrained PPT Generation

## Overview

This skill defines a constrained enterprise PowerPoint production pipeline. It is not a general-purpose “make a pretty PPT” workflow. Its purpose is to extract approved writing, page-structure, and visual rules from high-quality reference decks, preserve those rules in an approved template package, and let ordinary users produce consistent decks from that package.

The governing separation is:

> **AI decides what to say; the approved template decides how it looks; the validator decides whether it may ship.**

The skill has two modes:

- **Template construction mode**: inspect multiple approved reference decks, infer candidate rules, and produce a template package for human approval. Do not treat inferred rules as approved until an owner reviews them.
- **Template production mode**: select an approved template, build the story, copy the template, fill registered placeholders/components, validate the result, and deliver only a passing deck.

Never silently fall back to free-form visual design.

## When to Use

Use this skill when:

- an enterprise needs PPT output aligned to an approved internal style;
- a user provides an approved `.pptx` template or a template package;
- multiple excellent reference decks must be reverse-engineered into reusable page and writing rules;
- generated decks must be checked for fonts, sizes, positions, slide counts, component usage, and overflow;
- consistency and auditability matter more than creative visual variation.

Do not use this skill for:

- unconstrained creative presentations;
- redesigning an existing corporate visual identity;
- inventing a new visual system when no approved template exists;
- silently choosing an approximate template when requirements do not match.

## Non-Negotiable Product Decisions

These are hard constraints, not suggestions:

1. **Template scope**: one approved template serves one clear scenario and narrative type, such as quarterly business reporting. Different scenarios use different templates. Shared brand assets may be reused, but page types and content rules remain scenario-specific.
2. **Component permission**: use a component whitelist with limited composition. The agent may fill registered placeholders and may copy, remove, or arrange registered components only where `rules.yaml` explicitly permits it. It may never invent a visual component or style.
3. **Template mismatch**: if no template matches, report the mismatch and stop. A near-match may be used only after the user explicitly selects it and the output must disclose the deviation.
4. **Overflow order**: compress wording first; then split across duplicated approved page types; then reject. Never reduce font size, alter line spacing, move or resize boxes, or overflow a placeholder.
5. **Reference evidence**: infer rules from multiple high-quality decks for the same scenario where possible. Separate corporate hard rules, scenario rules, and individual stylistic accidents. Human approval is required before a package becomes production-approved.
6. **Output gate**: a failed validator means `REJECTED`, not “best effort delivered.”

## Template Package Contract

A production template is a closed visual system in its own directory. Recommended structure:

```text
ppt_skill/
├── workflow.md
├── system_rules.md
├── templates/
│   └── <scenario-name>/
│       ├── template.pptx
│       ├── visual_spec.yaml
│       ├── layout_schema.json
│       ├── rules.yaml
│       ├── assets/
│       └── README.md
└── validator/
    └── check_deck.py
```

Required artifact meanings:

- `template.pptx`: approved base deck containing masters, layouts, registered text/image/chart placeholders, headers, footers, and page types. Treat it as immutable input; copy it before editing.
- `visual_spec.yaml`: approved fonts, sizes, weights, colors, alignment, spacing, contrast, and brand assets.
- `layout_schema.json`: page types and their named elements, roles, bounds, allowed counts, and permitted composition operations.
- `rules.yaml`: content limits, page sequencing, overflow policy, component whitelist, required fields, and forbidden operations.
- `assets/`: template-owned logos, icons, images, or other assets. Do not fetch or invent replacements without an explicit rule.
- `check_deck.py`: deterministic structural/style validator. It must produce actionable errors and a non-zero exit status on failure.

Use stable semantic IDs such as `title`, `key_message`, `body`, `metric_1`, `problem_card_1`, `image_placeholder`, and `source_note`. Do not rely only on visual order or shape index.

## Mode A — Constructing an Approved Template

### A1. Establish evidence and scope

Collect the scenario name, intended audience, page purpose, owner, and all reference decks. Prefer at least three strong examples from the same scenario. Record which decks are approved exemplars and which are merely informative.

Completion criterion: a manifest exists listing every input deck, its scenario, approval status, and any known exceptions.

### A2. Extract candidate rules

Inspect the reference decks for:

- page-type inventory and recurring sequence;
- title, conclusion, evidence, explanation, and source placement;
- text density and content length ranges;
- repeated component geometry and counts;
- font family, size, weight, color, alignment, line spacing;
- margins, grids, chart/image treatment, headers, footers, and branding;
- conditions for showing images, data, comparisons, timelines, or callouts.

Classify each observation as:

- **hard corporate rule**: must hold across scenarios;
- **scenario rule**: applies to this template only;
- **candidate preference**: observed but not sufficiently evidenced;
- **sample accident**: do not encode.

Completion criterion: every proposed rule cites its source deck(s), confidence, and classification.

### A3. Encode the package

Create or update the immutable base template and the three machine-readable specifications. Define all legal page types and all legal components. Explicitly encode:

- whether a page may be duplicated;
- whether a component may be copied, removed, or reordered;
- minimum and maximum counts;
- text character/line limits;
- required and optional placeholders;
- allowed data visualizations and image treatment;
- exact validation tolerances, if any.

Do not encode a rule merely because it is convenient to implement. If a rule cannot be checked, label it as a human-review rule rather than pretending it is deterministic.

Completion criterion: a fresh package can be inspected without consulting the reference decks to determine what operations are legal.

### A4. Obtain approval

Present the inferred page inventory and rule summary to the template owner. Mark the package `draft` until approved. Production mode must refuse draft packages unless the user explicitly requests a draft run.

Completion criterion: package status, approver, date, version, and known exceptions are recorded in `README.md` or the manifest.

## Mode B — Producing a Deck

### B1. Parse the request and build the story

First decide content only. Convert the request into a narrative outline appropriate to the selected scenario, for example:

1. context and objective;
2. key conclusion;
3. problem or gap;
4. analysis/evidence;
5. response or solution;
6. implementation;
7. results;
8. next steps and risks.

The sequence is not universal: use the template's registered narrative rules. Identify the intended takeaway for each slide and distinguish conclusions from evidence and explanation.

Completion criterion: every planned slide has a purpose, takeaway, page type, content fields, and evidence/source requirements before any visual editing begins.

### B2. Match an approved template

Load the available template manifests and compare scenario, audience, narrative type, required page types, and content modalities. Select only an approved exact match by default.

If there is no exact match, stop with a mismatch report. If the user explicitly authorizes a near-match, record the selected template, mismatch dimensions, and expected risks before proceeding. Never conceal this exception.

Completion criterion: the selected template ID, version, approval status, match rationale, and exception status are recorded.

### B3. Load rules before editing

Read `visual_spec.yaml`, `layout_schema.json`, and `rules.yaml` before touching the deck. Treat these files as authoritative over model experience, generic PPT advice, or visual intuition.

Forbidden operations include, unless explicitly whitelisted:

```text
change_font
change_font_size
change_color
create_new_textbox
move_textbox
resize_textbox
change_layout
add_background
redesign_slide
create_new_visual_style
add_unregistered_shape
add_unregistered_image_container
modify_master
modify_theme
```

Completion criterion: the planned operations are a whitelist-valid operation list; any forbidden operation is removed or the run is rejected.

### B4. Copy, then fill

Copy `template.pptx` to a new working deck. Never edit the approved source in place. Use only semantic placeholders and registered components. Preserve masters, themes, layouts, geometry, and styles.

The agent may:

- replace placeholder text;
- insert approved data into a registered chart/table slot;
- insert an approved asset into a registered image slot;
- apply only registered component variants;
- duplicate or remove registered components when the rules allow it;
- duplicate an approved page type when the rules allow splitting.

The agent may not create a new textbox, shape, slide archetype, background, or visual style from scratch.

Completion criterion: a before/after inventory shows that every changed object is a registered placeholder or explicitly permitted component operation.

### B5. Handle content density and overflow

For each placeholder, check character count, line count, estimated fit, and semantic completeness. When content does not fit:

1. rewrite more concisely without losing the claim, qualifier, or evidence;
2. move excess content to another instance of the same approved page type or duplicate an approved page;
3. reject with a precise overflow report if the approved package cannot express the content.

Do not solve overflow by shrinking typography, changing line spacing, moving boxes, or hiding content. Never omit a material caveat merely to make a page pass.

Completion criterion: every placeholder passes fit checks, or every rejection names the slide, placeholder, limit, and unresolved content.

### B6. Validate deterministically

Run the package validator against the generated deck. At minimum validate:

- file opens and is a valid PPTX;
- expected template/version and masters remain intact;
- fonts, sizes, weights, colors, alignment, and spacing match the spec;
- semantic placeholder bounds and roles are unchanged;
- slide/page type and slide count are legal;
- required placeholders are populated;
- no unregistered shapes or components were added;
- component counts and permitted operations are respected;
- text does not overflow or become empty where prohibited;
- approved assets, sources, and page-level requirements are present.

Use tolerances only where the package defines them. Produce errors like:

```text
REJECTED
Slide 4 / body: expected Microsoft YaHei 18 pt; found Arial 20 pt.
Slide 5: unregistered shape id=shape_19.
Slide 6 / problem_card_3: text exceeds approved 4-line limit.
```

Completion criterion: the validator exits successfully and emits `PASS`; otherwise do not deliver the deck.

### B7. Perform visual QA

Deterministic checks cannot guarantee visual quality. Render the deck to images or PDF when a rendering tool is available and inspect every slide for clipping, missing glyphs, broken charts, unexpected blank space, unreadable contrast, and asset corruption. Rendering is a QA step, not permission to redesign.

Use a cross-platform font stack where the format permits it. Do not assume a single platform font exists. If required corporate fonts are unavailable, report the missing dependency rather than silently substituting a visually incompatible font.

Completion criterion: each slide has either a recorded visual inspection result or an explicit tooling limitation; any visual defect blocks delivery.

### B8. Deliver with an audit record

Deliver the PPTX only after structural and visual checks pass. Include a compact audit record containing template ID/version, source reference, generated file path, validator result, visual QA result, and any user-approved exceptions.

Completion criterion: the output file exists, reopens successfully, and the audit record points to the exact file and validation run.

## Validator Design Rules

The validator is a gate, not a beautifier. It should be deterministic, explain failures, and avoid false confidence.

Prefer these checks:

```python
assert deck_template_version == expected_version
assert shape_id in registered_ids
assert actual_font == expected_font
assert abs(actual_size - expected_size) <= allowed_tolerance
assert actual_bounds == expected_bounds
assert actual_slide_type in allowed_page_types
assert actual_shape_count <= allowed_shape_count
assert not text_overflows(placeholder)
```

For checks that are inherently visual or semantic, report `REVIEW_REQUIRED` rather than returning a misleading `PASS`. A deck is shippable only when all blocking checks pass and no required review remains unresolved.

Keep validator output machine-readable where practical, for example JSON with `status`, `errors`, `warnings`, and `review_required`, alongside a human-readable summary.

## Content Rules

The skill may reorganize, compress, and improve language, but content claims remain the user's responsibility. Do not invent metrics, sources, approvals, customer results, or causal explanations. Mark missing evidence explicitly and ask for it when it is necessary to complete a required page.

A good slide has one primary takeaway. Prefer a conclusion-led title when the template's writing rules support it. Put evidence in registered evidence areas, explanations in registered explanation areas, and sources in registered source areas. Do not force every topic into a generic “title + bullets” page if the template defines a more specific page type.

## Cross-Platform Implementation Rules

- Use `pathlib.Path`, not hard-coded path separators.
- Use Python scripts rather than shell scripts in `scripts/`.
- Use `tempfile` for temporary output; never assume `/tmp`.
- Invoke subprocesses with argument lists, not shell-constructed strings.
- Discover rendering tools with `shutil.which()` or an explicit environment variable.
- Do not assume `python`, Bash, LibreOffice, PowerPoint, or a particular browser exists.
- Treat fonts as environment capabilities. Use a documented cross-platform fallback stack only when the approved package permits fallback; otherwise fail clearly.
- If the skill package contains tests, include `tests/test_cross_platform.py` covering path handling, shell-script absence, hard-coded temporary paths, and documentation of Windows behavior.

## Failure Policy

Stop and report instead of improvising when:

- no exact template matches and no explicit near-match authorization exists;
- the selected package is missing, draft-only, corrupt, or internally inconsistent;
- required content or evidence is missing;
- content cannot fit after approved compression and splitting;
- an operation would require a forbidden visual change;
- fonts or rendering dependencies are unavailable and affect correctness;
- validator or visual QA fails.

A failure report must identify the stage, artifact, exact rule, observed value, expected value, and the smallest acceptable next action.

## Common Pitfalls

1. **Treating a single exemplar as the company standard** — use multiple same-scenario decks and require owner approval.
2. **Letting “有限组合” become free-form design** — every copy/remove/reorder operation must be registered in `rules.yaml`.
3. **Choosing a near-match silently** — stop unless the user explicitly authorizes it and log the deviation.
4. **Solving overflow with smaller text** — compress, split using approved pages, or reject.
5. **Editing the source template in place** — always copy it and verify the source hash or metadata remains unchanged.
6. **Checking only fonts and colors** — validate semantic roles, bounds, counts, content fit, required fields, and assets too.
7. **Equating validator PASS with visual quality** — render and inspect; mark inherently visual checks for review.
8. **Inventing business evidence** — never fabricate metrics, sources, or outcomes to fill a placeholder.
9. **Assuming Microsoft YaHei or another single font exists everywhere** — detect availability and report incompatibility.
10. **Using shape indexes as stable IDs** — shape order changes; use semantic IDs and explicit metadata.

## Verification Checklist

- [ ] The scenario and template version are identified.
- [ ] The template is approved, or draft/exception status is explicit.
- [ ] The story outline assigns every slide a takeaway and approved page type.
- [ ] `visual_spec.yaml`, `layout_schema.json`, and `rules.yaml` were loaded before editing.
- [ ] The source template was copied, not modified in place.
- [ ] Every edit is a placeholder fill or a whitelist-approved component operation.
- [ ] No forbidden visual operation occurred.
- [ ] Overflow was handled only by compression, approved splitting, or rejection.
- [ ] Required content, sources, and assets are present.
- [ ] Structural validator returned `PASS` with no blocking errors.
- [ ] Rendered slides received visual QA, or limitations are explicitly recorded.
- [ ] Output path and audit record are verifiable.
