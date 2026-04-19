---
description: Rules and standards applied during design system audits. Referenced automatically by all audit checks.
---

# Skill: Audit Standards

## Purpose
Define the rules and standards applied during design system audits. All audit checks reference this skill automatically.

## Pre-Check: Node Exclusions

Before running any check on any frame, identify all nodes of type SECTION in the layer tree. Section container nodes are top-level organisational wrappers in Figma, not design elements. Do not evaluate them for any check type -- spacing, variables, accessibility, typography, component integrity, or layer hygiene. Skip section nodes entirely and proceed to their children.

## Pre-Audit Context

Before running any checks, ask the user the following two questions and wait for both answers before proceeding:

1. Are there any elements, layers, or frames you want excluded from this audit? (Type NONE if not applicable)
2. Are there any known exceptions or intentional deviations from DS standards I should be aware of? (Type NONE if not applicable)

Store both answers for the duration of this audit session.

Exclusions: Before evaluating any node, walk its full ancestor chain from the node up to the top-level frame. If the node itself or any ancestor at any level appears in the exclusion list, skip the node entirely across all check types without flagging. Do not evaluate any child of an excluded node at any depth. The exclusion covers the named node, its entire descendant subtree, and any node whose ancestor chain passes through the excluded node.
Exceptions: When a violation matches a listed exception, do not flag it as a violation. Instead, add a line in the report: "Known exception: [user's description]"

## Spacing
Spacing grid is derived from the DS Brand Tokens database -- specifically the scale collection. Do not hardcode any grid value. Read the scale values from Notion, calculate the base unit, and validate all spacing against multiples of that unit.

Tolerance: +/- 1px is acceptable.

## Page Margins
Page margin values are stored in config.json under margins. Apply the values defined there per breakpoint. If config.json is not found, stop and ask the user to run /setup-ds first.

## Token Tier Compliance
A valid token usage chain is: Brand Token → Alias Token → Mapped Token → Component fill.

Violations:
- Any component fill bound directly to a Brand Token is a violation
- Any component fill bound directly to an Alias Token is a violation
- Any hardcoded value is a violation
- Any missing variable binding is a violation

## Accessibility -- WCAG Standard

Accessibility checks are mandatory and must run on every audit without exception. Do not skip or defer accessibility checks regardless of which other checks are running.

The accessibility standard to apply is stored in config.json under wcag_standard. Valid values: WCAG_2_1_AA, WCAG_2_2_AA, WCAG_2_2_AAA.

WCAG 2.1 AA minimums:
- Normal text contrast: 4.5:1
- Large text contrast (18pt+ regular, 14pt+ bold): 3:1
- UI components and icons: 3:1 against their direct interactive background. The direct interactive background is the fill of the immediate parent container that is part of the same interactive element, or the container the component is placed directly inside and which has an interactive function. Do not check contrast between two decorative background layers where neither layer is interactive and neither conveys meaning on its own.
- Surface containers, card backgrounds, table backgrounds, and panel backgrounds whose only function is to visually group or separate content are decorative. Do not apply the 3:1 check to contrast between these surfaces and the page background behind them. If flagging is warranted, add it as a LOW note with the label SURFACE CONTRAST rather than a CRITICAL or HIGH violation.
- Touch targets (mobile): 44x44px

WCAG 2.2 AA adds:
- Focus indicators must have 3:1 contrast ratio against adjacent colors
- Focus indicators must have minimum area of perimeter x 2px
- Touch targets: 24x24px minimum with no overlap (upgraded from 2.1)

WCAG 2.2 AAA adds:
- Normal text contrast: 7:1
- Large text contrast: 4.5:1

For contrast violations, always cross-reference the Notion DS token databases and suggest the closest DS token that would pass.

Scan depth: check text colors and fill values at all depths within every component instance. This includes nested component instances, child frames inside variants, and text layers at any level of nesting. Do not stop at the top-level instance -- descend into the full layer tree of each component to catch violations on nested elements such as placeholder text inside an input field that is itself inside a compound component.

## Typography
Valid font families, line height multipliers, and text styles are defined in the Notion Text Styles and Responsive Tokens databases. Do not hardcode any values. Read from Notion at the start of each audit.

## Component Integrity
The component library source of truth is the Figma DS file defined in config.json under ds_file_url. All component instances in the audited file must trace back to this library.

Violations:
- Detached instances -- an instance is detached only when Figma reports its node type as FRAME or GROUP instead of INSTANCE. Do not flag a component as detached based on name comparison, property value differences, or missing Notion database matches. A live INSTANCE node with property overrides is not detached.
- Manual property overrides outside defined variants
- Custom components that duplicate existing DS components

## Component States
Every interactive component must have the following state values present somewhere in its variant set:

* default / Default
* hover / Hover
* focus / Focus
* active / Active / pressed / Pressed
* disabled / Disabled
* error / Error -- where applicable

State detection rules:
- Scan the VALUES of all variant properties across the full variant set. Do not look for a property named "State" -- the property carrying state values can be named anything: Status, State, Mode, Interaction, Type, or any other name.
- A component passes the check for a given state if that state value appears anywhere in the full variant set, regardless of which property carries it.
- Only flag a component as missing a state if that state value is absent from the entire variant set.
- Casing differences (Default vs default) are acceptable and must not be flagged as inconsistencies or missing states.

## Responsive Coverage
Required breakpoints are derived from the Responsive Tokens database -- desktop, tablet, and mobile. Check that designs are present for all breakpoints defined in the DS.

## Color Independence
Information must not be conveyed by color alone. Any element that uses color as the only differentiator must have a secondary indicator (icon, pattern, label, or text).

## Layer Hygiene
- Flag any layer using a default Figma auto-generated name
- Flag any hidden layer
- Flag any empty group or frame
- Flag any layer more than 6 levels deep

## Reporting Format
Group all violations by priority level first, then by frame within each priority group.

For each priority group that has violations, output a section header:

- CRITICAL
- HIGH
- MEDIUM
- LOW

Omit priority groups with no violations entirely.

Within each priority group, group violations by frame. For each frame, output the frame name as a subheading. Then for each violation:

Frame: [frame name] > Component: [nearest named parent component] > Element: [element name]
- Current: [current value]
- Expected: [expected value]
- Note: [reason if not immediately obvious]

If the element has no named parent component and sits directly in the frame, omit the Component segment: Frame: [frame name] > Element: [element name]

Leave one blank line between each violation block. Leave two blank lines between frames.

Every hex color value anywhere in the output must be preceded by its color swatch indicator.

Priority levels:

- CRITICAL: token tier violations, WCAG contrast failures
- HIGH: missing component states, responsive coverage gaps, detached instances
- MEDIUM: typography violations, color independence issues
- LOW: layer hygiene, naming issues

End with a summary count per priority level.

After the full report has been output in chat, save the complete report to a file in the current working directory. File name format: audit-DD-MM-YYYY-HH-MM.md using the current date and time. Confirm the saved file name and path after saving.
