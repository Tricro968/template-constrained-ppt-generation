# template-constrained-ppt-generation

A Hermes skill for producing enterprise PowerPoint decks from approved, closed visual template systems.

## What it does

- Reverse-engineers multiple approved reference decks into candidate corporate and scenario rules.
- Separates content decisions from visual decisions.
- Restricts generation to registered placeholders and whitelisted components.
- Handles overflow by concise rewriting, approved page duplication, or rejection.
- Requires deterministic validation and rendered visual QA before delivery.

## Two operating modes

1. **Template construction** — analyze same-scenario reference decks, classify inferred rules, encode a template package, and require human approval.
2. **Template production** — match an approved template, build the story, fill legal placeholders/components, validate, render, inspect, and deliver.

## Installation

Copy this repository into your Hermes skills directory:

```text
~/.hermes/skills/productivity/template-constrained-ppt-generation/
```

Start a new Hermes session so the skill index reloads.

## Usage

Ask Hermes explicitly, for example:

```text
Use template-constrained-ppt-generation in template construction mode.
Analyze these approved quarterly-report PPT files and create a draft template package.
```

Or:

```text
Use template-constrained-ppt-generation in template production mode.
Generate a quarterly report from this approved template package and these business inputs.
```

For production, provide an approved package containing at least:

```text
template.pptx
visual_spec.yaml
layout_schema.json
rules.yaml
```

The current repository defines the workflow and behavioral contract. Enterprise templates, brand assets, and validators are organization-specific and are not bundled.

## Requirements

- Hermes Agent with skill support.
- An approved PPT template package for production mode.
- A PPT manipulation/rendering toolchain available to Hermes when producing actual files.
- Required corporate fonts installed or otherwise available; the skill must report missing fonts rather than silently substitute them.

No Python or Node package is required merely to load this skill.

## Supported platforms

Linux, macOS, and Windows. Actual PowerPoint rendering capabilities depend on the tools installed in the running environment.

## Documentation

See [SKILL.md](SKILL.md) for the complete workflow, constraints, failure policy, and verification checklist.

## License

MIT
