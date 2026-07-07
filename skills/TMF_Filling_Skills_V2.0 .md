
# TMF Filling Skills V2.0

## Purpose
TMF Filling Skills V2.0 is designed to identify TMF document types from uploaded PDFs, perform rule-based content recognition, segment documents at the content level, apply special-case combination rules, generate compliant TMF names, and output a structured TMF ZIP package.

The workflow is built for mixed, scanned, rotated, duplicated, or incomplete PDF packages where a single PDF may contain multiple TMF document types and special grouping dependencies.

---

## Rule Inputs
The workflow relies on:
1. TMF naming rules
2. special-case grouping / merging rules
3. output validation and correctness criteria

The naming rule source is expected to define:
- Document Type
- Document Subtype
- Document Classification
- Expected Document Name
- Short Name Requirement
- Version Name

The TMF taxonomy includes categories such as:
- IRB or IEC Submission
- IRB or IEC Approval
- IRB or IEC Composition
- Protocol Signature Page
- Protocol Amendment Signature Page
- Relevant Communications
- IP Accountability Documentation
- IP Return Documentation
- IP Certificate of Destruction
- IP Storage Condition Documentation
- Non-IP Shipment Documentation
- Shipment Records

---

## Core Principles
1. Do not modify the original PDF content.
2. Ignore watermark noise during analysis only.
3. Do not mechanically split by page.
4. Use content boundaries and TMF rules to define files.
5. Produce actual output files, not just structural descriptions.
6. Validate correctness through explicit criteria, not subjective judgment.

---

## Hard Constraints

### 1. Mandatory Coverage
The full PDF must be processed.
Every page must have a destination:
- classified output,
- unclassified / exception,
- or duplicate removal.

If any page is not accounted for, successful output must be blocked.

### 2. No Early Stop
Blur or unreadable pages must not stop the workflow.
All later pages must still be analyzed.

### 3. Dedup Mandatory
The workflow must detect and handle:
- page-level duplicates,
- file-level duplicates.

Deletion reasons must be recorded.

### 4. Secondary Segmentation
The same document type does not automatically mean the same file.
The workflow must continue checking:
- title differences
- version differences
- date differences
- signer differences
- page continuity

### 5. Orientation as a Gate
Orientation normalization and reading-order normalization are mandatory preconditions.
Downstream logic must not continue until they are completed.

---

## Processing Steps

### Step 1 — Load Rules
Load:
- TMF naming rules
- special-case grouping rules
- naming templates

Outputs:
- document type dictionary
- naming templates
- special-case rule map

---

### Step 1.1 — PDF Preprocessing
Before content analysis:
- ignore scan-app watermarks,
- ignore fixed promotional headers and footers,
- ignore repeated transparent OCR overlays.

Watermark content must not be used for:
- keyword matching,
- document type recognition,
- naming field extraction.

---

### Step 1.2 — Orientation and Reading-Order Normalization
Before classification or segmentation:
- correct upside-down pages,
- correct landscape pages,
- correct 90° / 270° rotations,
- reconstruct multi-page documents into natural reading order.

Use evidence such as:
- title
- page numbering
- header/footer
- shipment ID
- tracking number
- sender / recipient
- date continuity

This is a hard gate.

---

### Step 2 — Content Analysis
Analyze page-level features such as:
- title and form name
- headers and footers
- keywords
- signature and date
- sender / recipient / submission target
- visit date / ethics meeting date
- shipment number
- tracking number
- destruction log number
- temperature logger number
- IWRS / RTSM / IRT references

Outputs:
- page feature table
- candidate document type
- grouping keys

---

### Step 3 — Type Matching
Use rules to map pages or page groups to TMF document classes.

If stable:
- assign the document type

If not stable:
- mark as `Unclassified`
- or `NeedUserReview`

If a page is type-identifiable but does not hit a special rule:
- it must still go through default segmentation
- it must not be dropped prematurely into `Unclassified`

---

### Step 3.1 — Deduplication
Mandatory:
- page-level duplicate detection
- file-level duplicate detection

Possible signals:
- image similarity
- OCR similarity
- repeated signature pages
- same title + version + structure

All removals must be logged.

---

### Step 4 — Content-Level Segmentation
Segmentation must be based on content boundaries rather than page numbers alone.

Boundary signals:
- document type change
- title change
- subject change
- version change
- date change
- signer change
- page continuity break
- structural break

Rules:
- continuous multi-page content of the same document should remain one file
- mixed types inside one PDF must be split by content boundary
- page-by-page splitting is not allowed as a default behavior

---

### Step 4.1 — Default Segmentation Rule
For content that does not hit a special rule:
- segment by `document type + continuous page range + content continuity`.

Fallback behavior:
- if type is identifiable but boundaries are not fully stable:
  - output the most reasonable range
  - mark `NeedUserReview = Yes`
- if type is not identifiable:
  - mark `Unclassified`
  - write it to the exception report

---

## Special Rules

### 5.1 EC Submission
If the same submission chain contains:
- Sponsor → Investigator
- Investigator → EC

Recognize both parts and merge them into one EC submission PDF in the correct order.

### 5.2 EC SAE Submission
If only Investigator → EC is present and the content clearly concerns SAE:
- classify as EC SAE submission
- output as one file

### 5.3 EC Approval + Composition
If an EC approval and a same-day committee composition record belong together:
- place the approval letter first
- place the composition document second
- merge them into one file

### 5.4 Follow-up Letter
If a follow-up package is detected:
- identify the email part
- identify the formal follow-up letter
- verify visit-date consistency
- place the email first and the letter second

### 5.5 IP Receipt Package
Group a receipt package using the strongest available keys:
1. shipment number / tracking number
2. CTSR consignment number
3. IWRS / RTSM shipment number
4. receipt date (supporting only)

Expected package components:
1. courier waybill
2. Clinical Trial Shipment  Receipt Form
3. temperature record
4. IWRS / RTSM receipt confirmation


If all required components are present:
- merge in the required order.

If any component is missing:
- mark `Incomplete`
- mark `NeedUserReview = Yes`
- record missing components in reports and logs

### 5.6 IP Return
At minimum, identify and group:
1. Clinical Trial Shipment Receipt Form
2. Destruction Form

Use tracking number and destruction log number to verify whether they belong to the same IP return event.

---

## Step 6 — Rule-Based Naming
Generate output names according to TMF naming templates.

Possible naming fields:
- type / subtype
- version
- date
- subject
- other rule-required fields

If fields are missing:
1. retry extraction,
2. use placeholders,
3. record missing fields in the reports.

---

## Step 6.1 — Blur / Unreadable Page Handling
Blur or unreadable pages:
- must not stop the workflow
- must be isolated as `Unclassified`
- must be recorded in exception outputs
- must not block later pages from being processed

---

## Step 7 — Structured Output
The workflow must produce:
- classified PDFs
- file-level metadata
- processing report
- missing/unclassified report
- log files
- ZIP package

Recommended output structure:

```text
TMF_Output.zip
├── Classified_Files/
├── Reports/
└── Logs/
```

## File-Level Metadata

Each output PDF should include:

- source page range
- document type
- applied rules
- confidence level (`High / Medium / Low`)
- need-user-review (`Yes / No`)

---

## Output Validation

Before final delivery, the workflow must validate the following:

### 1. Page Integrity
- input page count must equal total accounted pages

### 2. Exception Visibility
- unclassified pages must be explicitly surfaced
- incomplete packages must be explicitly surfaced

### 3. Mandatory Logic Execution
- orientation normalization completed
- deduplication completed
- secondary segmentation completed

### 4. Blur Handling
- blur pages must be isolated only
- later-page processing must still be completed

If any validation fails:

- block output
- mark the workflow as `ProcessingFailed`

---

## State Management

Recommended main states:

- `Init`
- `RuleLoaded`
- `Preprocessed`
- `Oriented`
- `Analyzed`
- `Typed`
- `Deduped`
- `Segmented`
- `SpecialGrouped`
- `Named`
- `Validated`
- `Delivered`

Recommended degraded or failure states:

- `Unclassified`
- `NeedUserReview`
- `Incomplete`
- `ProcessingFailed`

---

## Exception Handling

Exception categories include:

- identification exceptions
- boundary exceptions
- quality exceptions
- completeness exceptions
- output exceptions

Examples:

- type cannot be determined
- multiple possible types compete
- segmentation boundary is unstable
- blur page exists
- package components are missing
- naming fields are missing
- file naming conflicts occur
- page coverage is not closed

---

## Evaluation and Correctness

### Coverage Correctness
Check:

```text
input page count = output-covered pages + duplicate-removed pages + exception-isolated pages
```

## Pipeline Correctness

Check whether:

- preprocessing happened
- orientation happened before classification
- dedup happened
- secondary segmentation happened
- special grouping happened before naming when required

---

## Type Correctness

Check whether outputs map to intended TMF classes rather than arbitrary labels.

---

## Segmentation Correctness

Check whether boundaries were determined by content and continuity instead of mechanical page splitting.

---

## Special-Rule Correctness

Check whether EC, Follow-up, IP Receipt, and IP Return logic used grouping keys and completeness checks rather than visual similarity.

---

## Naming Correctness

Check whether output names follow rule-defined patterns and whether missing fields are explicitly surfaced.

---

## Auditability

Each output file should include:

- source page range
- document type
- applied rules
- confidence level
- need-user-review flag

Correctness is therefore established by explicit validation criteria rather than subjective confidence alone.

---

## Boundary Cases

### 1. Type is recognizable but segmentation boundary is not stable
- output the most reasonable range
- mark `NeedUserReview = Yes`

### 2. Special-case package is incomplete
- output recognized parts
- mark `Incomplete`

### 3. Blur page exists
- isolate as `Unclassified`
- continue processing all later pages

### 4. Same type but different instances
- use version, date, signer, and page continuity to separate them

---

## Failure Modes

The workflow should be considered `Partial` or `Fail` if any of the following occurs:

- page coverage is not closed
- preprocessing or orientation normalization was skipped
- processing stopped early
- deduplication was skipped
- segmentation was page-mechanical
- special-case grouping used no reliable key matching
- naming ignored rule templates
- missing or unclassified content was hidden while claiming full completion

---

## Output Deliverables

Recommended deliverables:

- `Classified_Files/*.pdf`
- `Classified_Files/*.metadata.txt`
- `Reports/Processing_Report.xlsx`
- `Reports/Missing_or_Unclassified_Items.xlsx`
- `Reports/Processing_Summary.txt`
- `Logs/Processing_Log.txt`
- `TMF_Output.zip`
