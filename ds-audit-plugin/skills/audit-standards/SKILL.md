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

Exclusions: Do not evaluate any element, layer, or frame the user listed. Skip them in every check type without flagging.
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
The accessibility standard to apply is stored in config.json under wcag_standard. Valid values: WCAG_2_1_AA, WCAG_2_2_AA, WCAG_2_2_AAA.

WCAG 2.1 AA minimums:
- Normal text contrast: 4.5:1
- Large text contrast (18pt+ regular, 14pt+ bold): 3:1
- UI components and icons: 3:1 against their direct interactive background. The direct interactive background is the fill of the immediate parent container that is part of the same interactive element, or the container the component is placed directly inside and which has an interactive function. Do not check contrast between two decorative background layers where neither layer is interactive and neither conveys meaning on its own.
- Touch targets (mobile): 44x44px

WCAG 2.2 AA adds:
- Focus indicators must have 3:1 contrast ratio against adjacent colors
- Focus indicators must have minimum area of perimeter x 2px
- Touch targets: 24x24px minimum with no overlap (upgraded from 2.1)

WCAG 2.2 AAA adds:
- Normal text contrast: 7:1
- Large text contrast: 4.5:1

For contrast violations, always cross-reference the Notion DS token databases and suggest the closest DS token that would pass.

## Typography
Valid font families, line height multipliers, and text styles are defined in the Notion Text Styles and Responsive Tokens databases. Do not hardcode any values. Read from Notion at the start of each audit.

## Component Integrity
The component library source of truth is the Figma DS file defined in config.json under ds_file_url. All component instances in the audited file must trace back to this library.

Violations:
- Detached instances
- Manual property overrides outside defined variants
- Custom components that duplicate existing DS components

## Component States
Every interactive component must have the following states present in the DS file:
- Default
- Hover
- Focus
- Active
- Disabled
- Error (where applicable)

When checking state coverage, scan across all variant properties in the full variant set of the component. Do not check per variant property axis in isolation. A component passes the states check if the state value appears anywhere in the full set of variant combinations, regardless of which variant property carries it. Only flag a component as missing a state if that state value is absent from the entire variant set.

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
For every violation, output:
[PRIORITY] Element name
Current: <current value>
Expected: <expected value>
Note: <brief reason if not obvious>

Priority labels:
- CRITICAL: token tier violations, WCAG contrast failures
- HIGH: missing component states, responsive coverage gaps, detached instances
- MEDIUM: typography violations, color independence issues
- LOW: layer hygiene, naming issues

Group violations by frame. End with a summary count per category.
