---

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

## Step 3 -- Connect to Figma

Ask as two separate questions and wait for each answer individually:

Question 1:
Paste the URL of the design file you want to audit.
Wait for response.

Question 2:
Type the exact page name to audit.
Wait for response.

If the page name provided does not exist in the file, output:
Page "[name]" not found. Available pages in this file:
[numbered list of page names]
Type the number or exact name of the page to audit.
Wait for response before proceeding.

Connect to the provided file and page via Figma Desktop Bridge.

If the connection fails, stop and output:
Desktop Bridge connection failed. Check that Figma Desktop is open, your file is open, and the Desktop Bridge plugin is running. Do not attempt any alternative connection method.

If the connection succeeds, output:
CONNECTED TO: [file name]
Page: [page name]

---

## Step 4 -- Discover frames

Read the specified page. List all top-level frames found:

FRAMES FOUND ON [page name]:
1. [frame name]
2. [frame name]
...

Which frames should I audit? Type the numbers separated by commas, or type ALL.

Wait for response before proceeding. Store the selected frames. Only audit the frames the user selected.

---

## Step 5 -- Select checks

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

Wait for response before proceeding. Store the selected check numbers for use in Step 7.

---

## Step 6 -- Pre-audit context

Output these two questions separately and wait for each answer individually:

Question 1:
Are there any elements, layers, frames, or components you want excluded from this audit? (Type NONE if not applicable)
Wait for response.

Question 2:
Are there any known exceptions or intentional deviations from DS standards I should be aware of? (Type NONE if not applicable)
Wait for response.

Store both answers and apply them throughout all checks following the exclusion and exception rules in the Audit Standards skill.

---

## Step 7 -- Run audit

Apply the Audit Standards skill for all checks.

Execution order: run each selected check type across all selected frames before moving to the next check type. Complete spacing on all frames, then token compliance on all frames, and so on. This ensures the report is organised by check type, not by frame.

Run only the checks selected in Step 5, in this order:
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
RUNNING: [check name] across [n] frames

After completing each frame within a check, output:
[check name] -- [frame name]: done

Do not pause between check types for user confirmation. Run all selected checks to completion before outputting the report.

If Desktop Bridge connection drops at any point, stop immediately and output:
AUDIT PAUSED
Disconnected during: [check name]
Last completed frame: [frame name]
Frames remaining: [list of frame names not yet checked for this check type]

To resume: reconnect the Desktop Bridge plugin in Figma and type RESUME.
On RESUME: restart the interrupted check from the first incomplete frame. Do not rerun frames already completed for this check type. Do not switch to any alternative data source.

---

## Step 8 -- Output report

After all checks complete, output the full report following the Reporting Format defined in the Audit Standards skill.

Report structure:
- Header block: file name, page name, checks run, frames audited, date and time
- One section per check type that was run
- Within each section: violations grouped by frame
- For each frame with violations: frame name as subheading, then each violation in the standard format
- For each frame with no violations: output "[frame name] -- no violations found"
- Summary: violation count per check type, combined total, count of frames with zero violations

---

## Step 9 -- Save report

After the full report is output in chat, save it to the current working directory.

File name format: audit-DD-MM-YYYY-HH-MM.md using the current date and time.

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
