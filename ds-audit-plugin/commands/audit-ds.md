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

## Step 2 -- Select checks

Before loading any data, ask which checks to run. This determines which databases need to be loaded.

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

Wait for response before proceeding. Store the selected check numbers.

---

## Step 3 -- Load DS reference data from Notion

Load only the databases required for the selected checks. Use this mapping:

- Spacing (1): Brand Tokens only
- Token tier compliance (2): Brand Tokens, Alias Tokens, Mapped Tokens
- Accessibility (3): Brand Tokens, Alias Tokens, Mapped Tokens
- Typography (4): Text Styles, Responsive Tokens
- Component integrity (5): Components
- Component states (6): Components
- Responsive coverage (7): Responsive Tokens
- Color independence (8): no database load required
- Layer hygiene (9): no database load required

Consolidate the required databases across all selected checks and load each database once only.

Output a loading confirmation after each database is read:
- Brand Tokens: [n] records loaded (or: not required for selected checks)
- Alias Tokens: [n] records loaded (or: not required)
- Mapped Tokens: [n] records loaded (or: not required)
- Responsive Tokens: [n] records loaded (or: not required)
- Text Styles: [n] records loaded (or: not required)
- Components: [n] records loaded (or: not required)

Then output:
DS DATA LOADED -- [n] databases loaded for selected checks. Ready to connect to Figma.

Do not query Notion again during the audit. All checks must use this session data exclusively.

If any required database query fails, stop and report which database failed. Do not proceed until all required databases are loaded successfully.

---

## Step 4 -- Define audit scope

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
  The DS reference data loaded in Step 3 is from the configured file. Auditing a different file against this data may produce inaccurate results.
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
- List all top-level frames found, one per line, with a sequential number
- If frames are organised under sections, output the section name as a subheading before its frames

Use this format exactly:

FRAMES FOUND ON [page name]:

[Section name]
1. [frame name]
2. [frame name]

[Section name]
3. [frame name]
4. [frame name]

If there are no sections, omit the section subheadings and list all frames sequentially.

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

Wait for response. If the user types YES, proceed to Step 5.
If the user types anything other than YES, ask them to clarify what needs changing, update the scope accordingly, output the revised scope, and ask for confirmation again. Repeat until the user confirms with YES.

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

Apply the Audit Standards skill for all checks. All checks must use the session data loaded in Step 3. Do not re-query Notion, Figma, or config.json during checks.

Execution order: run each selected check type across all scoped frames or nodes before moving to the next check type.

Run only the checks selected in Step 2, in this order:
1. Spacing
2. Token tier compliance
3. Accessibility
4. Typography
5. Component integrity
6. Component states
7. Responsive coverage
8. Color independence
9. Layer hygiene

Node tracking: maintain a running list of evaluated nodes per check type throughout the audit. For every frame or node evaluated during a check, add it to the evaluated list for that check. For every frame or node in scope that was not reached, add it to the not-reached list for that check. These lists are used in Step 8.

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

If the user types RESTART: begin the full audit again from Step 4. All session data from Step 3 is still valid and does not need to be reloaded.

Note: RESUME and RESTART only work within the same session. If this session has been closed and reopened, the audit state is lost. Begin from Step 1.

---

## Step 7 -- Output report

After all checks complete, output the full report following the Reporting Format defined in the Audit Standards skill.

If the audit scope spans multiple pages, output one complete report per page, each with its own header and summary.

Report structure per page:
- Header block: file name, page name, checks run, frames or nodes audited, date and time
- One section per check type that was run
- Within each section: violations grouped by priority first, then by frame within each priority group
- For each frame or node with violations: name as subheading, then each violation in the standard format
- For each frame or node with no violations: output "[name] -- no violations found"
- Summary: violation count per priority level and per check type, combined total, count of frames or nodes with zero violations

---

## Step 8 -- QA coverage check

After the report is output in chat, run a self-audit before saving. Use the node tracking lists maintained during Step 6.

For each check type that was run, output a coverage block:

CHECK: [check name]
- Frames or nodes in scope: [list from Step 4]
- Frames or nodes evaluated: [list from Step 6 tracking]
- Nodes not reached: [any in-scope frames or nodes not evaluated -- or NONE]

After all coverage blocks, output:

COVERAGE SUMMARY
- Checks with full coverage: [list]
- Checks with gaps: [list, or NONE]

Then output:

QA OPTIONS
Type COMPLETE to accept the report as-is and save.
Type FILL [check type] to rerun that check on missed nodes only and merge results before saving.
Type FILL ALL to rerun all checks with gaps on their missed nodes before saving.

Wait for response before proceeding.

On COMPLETE: proceed to Step 9.

On FILL [check type]:
- Rerun the named check on the nodes listed as not reached for that check only
- Do not rerun nodes already evaluated
- Merge the new findings into the existing report in the correct section
- Update the node tracking list for that check
- If Desktop Bridge drops during fill, stop and output: FILL PAUSED -- reconnect Desktop Bridge and type RESUME FILL to continue
- Output: FILLED: [check name] -- [n] additional nodes evaluated, [n] new violations found
- Output a refreshed coverage summary showing updated gap status across all checks
- Return to the QA OPTIONS prompt and wait for response

On FILL ALL:
- Rerun each check that has gaps, on its missed nodes only, in the same order as Step 6
- Do not rerun nodes already evaluated
- Merge all new findings into the existing report in their correct sections
- Update node tracking lists for all filled checks
- If Desktop Bridge drops during fill, stop and output: FILL PAUSED -- reconnect Desktop Bridge and type RESUME FILL to continue
- Output: FILLED ALL -- [n] checks supplemented, [n] additional nodes evaluated, [n] new violations found
- Proceed directly to Step 9

---

## Step 9 -- Save report

After the QA gate is resolved, read the current date and time at the moment of writing the file -- not from session start. Use this value for both the filename and the timestamp in the report header.

Save the report to the current working directory.

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
- Do not re-query Notion, Figma, or config.json during checks -- use session data from Step 3 exclusively
- If anything fails, stop and report the failure -- do not improvise a workaround
