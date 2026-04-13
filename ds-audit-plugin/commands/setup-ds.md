---
description: One-time guided setup that connects the plugin to your Notion DS documentation and Figma DS file, saving configuration to ${CLAUDE_PLUGIN_DATA}/config.json.
---

# Command: /setup-ds

## Purpose
One-time guided setup for the DS Audit Plugin. Collects DS-specific configuration from the user and saves it to ${CLAUDE_PLUGIN_DATA}/config.json. All other commands depend on this file.

## When to run
Run once when first installing the plugin. Re-run any time the DS structure changes significantly -- new Notion databases, new Figma file, different platform scope.

## Pre-conditions
- Notion must be connected
- Figma Console MCP must be connected
- Desktop Bridge plugin must be running in Figma with the DS file open

---

## Step 1 - Welcome

Output:

---
DS AUDIT PLUGIN SETUP

This setup connects the plugin to your Notion DS documentation and Figma DS file.
It takes about 5 minutes and only needs to be done once.

Before continuing, confirm:
- Notion is connected (run /mcp to check)
- Your DS Figma file is open in Figma Desktop
- The Desktop Bridge plugin is running in Figma

Type START to begin or EXIT to cancel.
---

Wait for response before proceeding.

---

## Step 2 - Discover Notion databases

Output:
STEP 1 OF 5 -- NOTION WORKSPACE

What is the name of the Notion teamspace where your DS documentation lives?

Wait for response.

Search Notion for all databases in the named teamspace.

List what you find:

DATABASES FOUND IN [teamspace name]:
[numbered list of database names with their types]

These databases will be used for all audit and update checks.
Do they look correct?

Type CONFIRM to continue.
Type RECHECK to search again if something looks wrong.

Wait for response before proceeding.

Store all discovered database IDs automatically. Do not ask the user to provide any IDs manually.

---

## Step 3 - DS Figma file

Output:
STEP 2 OF 5 -- FIGMA DS FILE

Paste the URL of your Figma DS file -- the file that contains your variables, styles, and components.

Wait for response. Extract and store the file key from the URL.

---

## Step 4 - Component location

Output:
STEP 3 OF 5 -- COMPONENT LOCATION

Where are your master components located?

1. All in one page inside the DS file -- I will ask for the page name
2. Spread across multiple pages inside the DS file -- I will scan and ask you to confirm which pages to include
3. In a separate dedicated Figma file -- I will ask for that file's URL

Type 1, 2, or 3.

Wait for response.

If 1: Ask for the page name.
If 2: Call figma_get_file_data with verbosity='summary' and depth=1 on the DS file. List all pages found and ask the user to confirm which ones contain master components.
If 3: Ask for the separate file URL.

Store the response as component_location and component_page or component_pages.

---

## Step 5 - Page margins

Output:
STEP 4 OF 5 -- PAGE MARGINS

What are your page margins per breakpoint?

You can enter custom values or choose from common presets:

Preset A -- Standard web: Desktop 64px, Tablet 32px, Mobile 16px
Preset B -- Compact: Desktop 48px, Tablet 24px, Mobile 16px
Preset C -- Wide: Desktop 80px, Tablet 40px, Mobile 24px

Type A, B, or C to use a preset, or enter custom values as:
Desktop: [value]px, Tablet: [value]px, Mobile: [value]px

Wait for response. Store the margin values.

---

## Step 6 - Accessibility standard and Platform

Output:
STEP 5 OF 5 -- ACCESSIBILITY STANDARD & PLATFORM

Which accessibility standard should audits apply?

1. WCAG 2.1 AA -- widely adopted baseline
2. WCAG 2.2 AA -- current W3C standard, required under European Accessibility Act
3. WCAG 2.2 AAA -- highest level, stricter contrast and sizing requirements

Type 1, 2, or 3.

Wait for response. Store as wcag_standard.

What platforms does your product target?

1. Web only
2. Mobile only (iOS and/or Android)
3. Both web and mobile

Type 1, 2, or 3.

Wait for response. Store as platform.

---

## Step 8 - Save config

Read the Brand Tokens database from Notion to derive the spacing scale.
Extract all numeric scale values, identify the base unit, and store it.

Then write config.json to ${CLAUDE_PLUGIN_DATA}/config.json with the following structure:

{
  "notion": {
    "brand_tokens_db": "",
    "alias_tokens_db": "",
    "mapped_tokens_db": "",
    "responsive_tokens_db": "",
    "text_styles_db": "",
    "components_db": ""
  },
  "ds_file_url": "",
  "ds_file_key": "",
  "component_location": "",
  "component_page": "",
  "spacing_scale": 0,
  "margins": {
    "desktop": 0,
    "tablet": 0,
    "mobile": 0
  },
  "wcag_standard": "",
  "platform": ""
}

Populate all fields from the responses collected in Steps 2-6 and the spacing scale derived from Notion.

If Claude Code asks for permission to write the file, approve it.

After writing, output:

---
SETUP COMPLETE

config.json saved to plugin data folder.

Your configuration:
- Notion databases: 6 connected
- DS file: [file name]
- Component location: [value]
- Spacing scale: [n]px base unit
- Page margins: Desktop [n]px / Tablet [n]px / Mobile [n]px
- Accessibility standard: [value]
- Platform: [value]

You are ready to run /audit-ds, /update-ds, and /handover-ds.
---

## Execution rules
- Never skip a step
- Never proceed past any input gate without a response
- Validate database IDs before saving
- If Notion connection fails at Step 8, report the error and ask user to reconnect before retrying
- If ${CLAUDE_PLUGIN_DATA}/config.json already exists, ask: Update existing config or start fresh?
