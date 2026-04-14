---
description: Defines how to read design system data from Notion databases. Referenced by all commands that need DS data.
---

# Skill: DS Reader

## Purpose
Define how to read design system data from the Notion databases created in Advanced Guide 2/5. All commands that need DS data reference this skill.

## Pre-conditions
- Notion must be connected
- config.json must exist in the current working directory with valid database IDs

## Reading config.json
At the start of every command, check for config.json in the current working directory. If not found, stop immediately and output:

"config.json not found in this folder. Make sure you opened your project folder in Cowork and that config.json is present. Run /setup-ds if you need to create it."

Do not proceed with any command until config.json is found.

If found, read config.json from the current working directory.

Output:

CONFIG FOUND

- Config file: [full path to config.json]
- Figma file: [ds_file_name value] -- [ds_file_url value]
- Notion workspace: [notion_workspace_name value] -- [notion_workspace_url value]
- Accessibility standard: [wcag_standard value]
- Platform: [platform value]

Type YES to proceed with this config, or SETUP to stop and run /setup-ds instead.

Wait for the user's response before doing anything else. If YES, proceed to extract all config values. If SETUP, stop and output: "Run /setup-ds to reconfigure your connection."

Extract the following values:
- notion.brand_tokens_db -- Brand Tokens database ID
- notion.alias_tokens_db -- Alias Tokens database ID
- notion.mapped_tokens_db -- Mapped Tokens database ID
- notion.responsive_tokens_db -- Responsive Tokens database ID
- notion.text_styles_db -- Text Styles database ID
- notion.components_db -- Components database ID
- ds_file_url -- Figma DS file URL
- component_location -- one_page / multiple_pages / separate_file
- component_page -- page name or URL if component_location is one_page or separate_file
- spacing_scale -- base unit in px (e.g. 8)
- margins.desktop -- page margin in px for desktop frames
- margins.tablet -- page margin in px for tablet frames
- margins.mobile -- page margin in px for mobile frames
- wcag_standard -- WCAG_2_1_AA, WCAG_2_2_AA, or WCAG_2_2_AAA
- platform -- web, mobile, or both

## Reading Brand Tokens
Query the Brand Tokens database.
For each record, extract: Name, Group, Value, Figma ID.
Use this to build a local token map for cross-referencing during audit.

## Reading Alias Tokens
Query the Alias Tokens database.
For each record, extract: Name, Group, Brand Token (relation), Figma ID.
Use this to build the alias chain map.

## Reading Mapped Tokens
Query the Mapped Tokens database.
For each record, extract: Name, Group, Light (relation), Dark (relation), Figma ID.
Use this to validate correct token usage in light and dark mode designs.

## Reading Responsive Tokens
Query the Responsive Tokens database.
For each record, extract: Name, Group, Desktop, Tablet, Mobile, Figma ID.
Use this to validate typography and spacing values per breakpoint.

## Reading Text Styles
Query the Text Styles database.
For each record, extract: Name, Font Family, Font Weight, Decoration, Figma ID.
Use this to validate typography compliance.

## Reading Components
Query the Components database.
For each record, extract: Name, Section, Type, Internal, Variant Count, Figma ID.
Use this to validate component usage in audited files.

## Spacing Scale Derivation
Read all records from the Brand Tokens database where Group contains 'scale'.
Extract all numeric values.
Identify the base unit (smallest non-zero value or greatest common divisor of the scale).
Store as spacing_scale in the session for use during spacing checks.

## Error Handling
- If any database query fails, report which database failed and stop
- If a relation cannot be resolved, log it and continue
- If config.json values are missing or malformed, stop and ask user to run /setup-ds again
