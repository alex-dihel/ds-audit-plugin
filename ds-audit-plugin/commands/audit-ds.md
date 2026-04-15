# Command: /audit-ds

## Purpose
Run a structured design system audit against a Figma design file. Checks selected audit types against DS standards stored in Notion, outputs a formatted report in chat, and saves it to a file.

## Pre-conditions
- config.json must exist in the current working directory
- Notion must be connected
- Figma Desktop Bridge must be running with the DS file open

## Absolute prohibitions
- Do not generate any artifact, React component, dashboard, or UI element under any circumstance
- Do not fall back to the Figma REST API if Desktop Bridge drops -- stop and report the disconnection instead
- Do not skip any step or combine steps
- Do not proceed past any input gate without an explicit response
- Do not read Figma or Notion data before config.json is confirmed

---

## Step 1 -- Config check

Read config.json from the current working directory following the DS Reader skill.

Output the confirmation block and wait for YES before proceeding.

If config.json is not found, stop and output:
config.json not found. Make sure you opened your project folder and that config.json is present. Run /setup-ds if needed.

---

## Step 2 -- Load DS reference data from Notion

Read the following databases from Notion using the IDs in config.json, following the DS Reader skill:
- Brand Tokens
- Alias Tokens
- Mapped Tokens
- Responsive Tokens
- Text Styles
- Components

Output a loading confirmation after each database is read:
- Brand Tokens: [n] records loaded
- Alias Tokens: [n] records loaded
- Mapped Tokens: [n] records loaded
- Responsive Tokens: [n] records loaded
- Text Styles: [n] records loaded
- Components: [n] records loaded

Then output:
DS DATA LOADED -- [total] records across 6 databases. Ready to connect to Figma.

Do not query Notion again during the audit. Use this loaded data for all checks.

If any database query fails, stop and report which database failed. Do not proceed until all six databases are loaded successfully.

---

## Step 3 -- Define audit scope

Output:
AUDIT SCOPE

Paste one or more Figma URLs to audit. You can paste:
- A page URL -- I will list the frames on that page and ask which to audit
- A frame, section, or element URL (containing a node-id) -- I will scope directly to that node without reading the full page
- Multiple URLs -- paste one per line

Wait for response before proceeding.

For each URL provided:
- Extract the file key
- Compare it against ds_file_key in config.json
- If the file key does not match, output:
  WARNING: This URL points to a different file than the one configured in config.json.
  Configured file: [ds_file_name from config.json]
  Pasted URL file key: [extracted file key]
  The DS reference data loaded in Step 2 is from the configured file. Auditing a different file against this data may produce inaccurate results.
  Type YES to proceed anyway, or paste a corrected URL.
  Wait for response before proceeding.

- Check whether a node-id parameter is present in the URL

If node-id is present:
- Connect directly to that node via Desktop Bridge
- Do not read the full page
- Add the node to the audit scope immediately
- Output: Scoped to: [node name] ([node-id]) on page [page name]

If no node-id is present:
- Connect to the file and read the specified page
- List all top-level frames found:

FRAMES FOUND ON [page name]:
1. [frame name]
2. [frame name]
...

Which frames should I audit? Type the numbers separated by commas, or type ALL.
Wait for response. Add selected frames to the audit scope.

If the page cannot be found, output:
Page not found. Available pages in this file:
[numbered list of page names]
Type the number or exact name of the page to audit.
Wait for response before proceeding.

After processing all URLs, group the audit scope by page:
- URLs pointing to the same page are combined into one audit with one report
- URLs pointing to different pages produce separate audits with separate reports

Output the final scope before proceeding:
AUDIT SCOPE CONFIRMED
[page name]: [list of frame or node names to audit]
[page name]: [list of frame or node names to audit]
...

Is this correct? Type YES to continue, or describe what needs changing.

Wait for response. If the user types YES, proceed to Step 4.
If the user types anything other than YES, ask them to clarify what needs changing, update the scope accordingly, output the revised scope, and ask for confirmation again. Repeat until the user confirms with YES.

---

## Step 4 -- Select checks

Output:
SELECT AUDIT CHECKS

Which checks should I run? Type the numbers separated by commas, or type ALL.

1. Spacing -- validates spacing values against the DS scale
2. Token tier compliance -- checks all fills are bound to the correct token tier
3. Accessibility -- WCAG contrast, text size, touch targets, focus indicators
4. Typography -- font families, line heights, text style bindings
5. Component integrity -- detached instances, manual overrides
6. Component states -- interactive state coverage
7. Responsive coverage -- breakpoint presence
8. Color independence -- information not conveyed by color alone
9. Layer hygiene -- naming, hidden layers, empty groups, depth

Wait for response before proceeding. Store the selected check numbers for use in Step 6.

---

## Step 5 -- Pre-audit context

Output these two questions separately and wait for each answer individually:

Question 1:
Are there any elements, layers, frames, or components you want excluded from this audit? (Type NONE if not applicable)
Wait for response.

Question 2:
Are there any known exceptions or intentional deviations from DS standards I should be aware of? (Type NONE if not applicable)
Wait for response.

Store both answers and apply them throughout all checks following the exclusion and exception rules in the Audit Standards skill.

---

## Step 6 -- Run audit

Apply the Audit Standards skill for all checks.

Execution order: run each selected check type across all scoped frames or nodes before moving to the next check type. Complete spacing on all scoped frames or nodes, then token compliance on all scoped frames or nodes, and so on. This ensures the report is organised by check type.

Run only the checks selected in Step 4, in this order:
1. Spacing
2. Token tier compliance
3. Accessibility
4. Typography
5. Component integrity
6. Component states
7. Responsive coverage
8. Color independence
9. Layer hygiene

Before starting each check type, output:
RUNNING: [check name] across [n] frames or nodes

After completing each frame or node within a check, output one of:
- [check name] -- [frame or node name]: done, [n] violations found
- [check name] -- [frame or node name]: done, no violations

Do not pause between check types for user confirmation. Run all selected checks to completion before outputting the report.

If Desktop Bridge connection drops at any point, stop immediately and output:
AUDIT PAUSED
Disconnected during: [check name]
Last completed frame or node: [name]
Frames or nodes remaining: [list of names not yet checked for this check type]

To resume: reconnect the Desktop Bridge plugin in Figma and type RESUME.
On RESUME: restart the interrupted check from the first incomplete frame or node. Do not rerun frames or nodes already completed for this check type. Do not switch to any alternative data source.

Note: RESUME only works within the same session. If this session has been closed and reopened, the audit state is lost. Type RESTART to begin the full audit from Step 3.

---

## Step 7 -- Output report

After all checks complete, output the full report following the Reporting Format defined in the Audit Standards skill.

If the audit scope spans multiple pages, output one complete report per page, each with its own header and summary.

Report structure per page:
- Header block: file name, page name, checks run, frames or nodes audited, date and time
- One section per check type that was run
- Within each section: violations grouped by frame or node
- For each frame or node with violations: name as subheading, then each violation in the standard format
- For each frame or node with no violations: output "[name] -- no violations found"
- Summary: violation count per check type, combined total, count of frames or nodes with zero violations

---

## Step 8 -- Save report

After the full report is output in chat, save it to the current working directory.

If the audit covers one page: one file.
If the audit covers multiple pages: one file per page.

File name format: audit-DD-MM-YYYY-HH-MM.md for a single page.
For multiple pages: audit-[sanitised-page-name]-DD-MM-YYYY-HH-MM.md per page.
Sanitise the page name by replacing spaces with hyphens and removing any characters that are not letters, numbers, or hyphens.

Output:
REPORT SAVED: [filename]

---

## Execution rules
- Follow every step in order
- Do not skip or combine steps
- Do not generate any visual artifact or UI under any circumstance
- Do not use the Figma REST API under any circumstance
- Do not proceed past any input gate without an explicit response
- If anything fails, stop and report the failure -- do not improvise a workaround
