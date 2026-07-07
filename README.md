# TMF Filling Skills V2.0

## Overview
TMF Filling Skills V2.0 is a rule-driven workflow for identifying, segmenting, naming, validating, and packaging TMF documents from mixed PDF inputs.

The goal is not to mechanically split PDFs page by page. Instead, the skill reconstructs a mixed PDF into a structured and auditable set of TMF files by combining:
- rule-based document classification,
- PDF preprocessing,
- page orientation and reading-order normalization,
- content-level segmentation,
- special-case grouping logic,
- rule-based naming,
- output validation,
- and audit-friendly reporting.

V2.0 introduces preprocessing and correctness evaluation as first-class system capabilities.

---

## Rule Foundation
This skill is aligned with the TMF naming and classification structure defined in the TMF rule workbook.

The rule workbook includes fields such as:
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

## What is new in V2.0
Compared with earlier versions, V2.0 adds:

### 1. PDF Preprocessing
- Ignores scan-app watermarks during analysis
- Ignores fixed promotional headers and footers
- Ignores repeated transparent OCR artifacts
- Prevents noise from affecting classification and naming

### 2. Orientation and Reading-Order Normalization
- Corrects upside-down, landscape, and rotated pages
- Reconstructs multi-page forms into human-readable order
- Treats orientation normalization as a mandatory gate before classification and segmentation

### 3. Hard Constraints
V2.0 introduces non-optional processing constraints:
- Full page coverage is mandatory
- Processing must not stop early on blur or unreadable pages
- Deduplication is mandatory
- Same document type still requires secondary segmentation if boundaries differ
- Orientation normalization must happen before downstream logic

### 4. Evaluation-First Correctness
Output is not considered correct just because files were generated.
Instead, correctness is evaluated through:
- page coverage,
- pipeline order,
- type correctness,
- segmentation correctness,
- special-rule correctness,
- naming correctness,
- and auditability.

### 5. Explicit Boundary Handling
Borderline cases are represented explicitly through states such as:
- `Unclassified`
- `NeedUserReview`
- `Incomplete`
- `ProcessingFailed`

---

## Key Features
- Rule-driven TMF document classification
- Rule-based document naming
- PDF preprocessing for watermark/noise suppression during analysis
- Orientation and reading-order normalization
- Content-level segmentation instead of page-based slicing
- Special handling for EC submission chains, Follow-up letters, IP receipt packages, and IP return packages
- Hard validation gates for coverage, deduplication, and segmentation quality
- Structured outputs with metadata, reports, logs, and ZIP packaging

---

## Processing Pipeline

### Step 0 — Hard Constraints
The following are mandatory gates:

#### Mandatory Coverage
Every input page must have an explicit outcome:
- classified output,
- unclassified / exception,
- or duplicate removal.

If any page has no destination, the workflow must not be considered successful.

#### No Early Stop
The workflow must continue processing even if blur, unreadable, or exception pages are encountered.

#### Dedup Mandatory
Both page-level and document-level duplicate detection must be performed.

#### Secondary Segmentation
The same document type does not automatically mean the same file.
Differences in title, date, version, signer, or page continuity must trigger further segmentation.

#### Orientation as a Gate
Orientation normalization and reading-order normalization must be completed before classification, segmentation, naming, or final validation.

---

### Step 1 — Rule Loading
Load and parse:
- TMF naming rules
- special-case grouping / merging rules (if available)

Outputs:
- type dictionary
- naming template dictionary
- special-rule triggers

---

### Step 1.1 — PDF Preprocessing
Before content analysis:
- ignore scan-app watermarks,
- ignore fixed promotional headers/footers,
- ignore repeated transparent OCR overlays.

Important:
- preprocessing affects analysis only,
- original content must not be modified,
- true document content must not be removed.

---

### Step 1.2 — Orientation and Reading-Order Normalization
Before recognition and segmentation:
- correct upside-down, landscape, and rotated pages,
- reconstruct natural reading order for multi-page forms,
- use title, header/footer, page numbering, shipment ID, tracking number, sender/recipient, and date continuity to determine adjacency.

This is a hard gate.

---

### Step 2 — Page-Level Content Analysis
Extract page-level features including:
- title and form name
- headers and footers
- keywords
- version
- signature and date
- sender / recipient / submission target
- visit date / ethics meeting date
- shipment / tracking / destruction log identifiers
- temperature logger identifiers
- IWRS / RTSM / IRT references

Outputs:
- page feature table
- document type candidates
- grouping key fields

---

### Step 3 — Type Matching
Map content to TMF document classes.

If matching is stable:
- assign a TMF type.

If matching is uncertain:
- mark as `Unclassified`
- or `NeedUserReview`

---

### Step 3.1 — Deduplication
Mandatory:
- page-level duplicate detection
- document-level duplicate detection

Duplicates may be detected using:
- image similarity
- OCR similarity
- signature-page repetition
- title + version + content structure similarity

All removals must be logged.

---

### Step 4 — Content-Level Segmentation
Segmentation must be based on document boundaries, not page numbers alone.

Boundary signals include:
- document type change
- title change
- subject change
- version change
- date change
- signer change
- page continuity break
- structural break in business logic

Principles:
- a continuous multi-page document should remain one file,
- a mixed-type PDF should be split by content boundary,
- single-page splitting is not allowed as a generic default.

---

### Step 4.1 — Default Segmentation Rule
For documents that do not hit a special rule:
- segment by `document type + continuous page range + content continuity`.

Fallback behavior:
- if type is clear but boundaries are imperfect: output the most reasonable range and mark `NeedUserReview = Yes`
- if type is not identifiable: mark `Unclassified`

---

### Step 5 — Special Rules

#### EC Submission
If both are found:
- Sponsor → Investigator
- Investigator → EC

Combine them into one EC submission file in the correct order.

#### EC SAE Submission
If only Investigator → EC is present and the content is clearly SAE-related:
- output as one EC SAE submission file.

#### EC Approval + Composition
If an EC approval package is meeting-based:
- identify the approval letter,
- identify the matching committee composition record,
- place the approval first and the composition record second.

#### Follow-up Letter
If a follow-up package is identified:
- detect the email component,
- detect the formal follow-up letter,
- verify that visit dates match,
- place the email first and the formal letter second.

#### IP Receipt Package
Group a receipt package by the strongest available keys:
1. shipment number / tracking number
2. CTSR consignment number
3. IWRS / RTSM shipment number
4. receipt shipment number
5. receipt date (supporting only)

Expected package components:
1. courier waybill
2. Clinical Trial Shipment Receipt Form
3. temperature record
4. IWRS / RTSM receipt confirmation
5. Local Depot Shipment Request

If all required components are present:
- merge in the required order.

If any component is missing:
- mark `Incomplete`
- mark `NeedUserReview = Yes`
- record missing components in reports and logs.

---

### Step 6 — Rule-Based Naming
Generate output names using TMF naming templates.

Naming fields may include:
- type / subtype
- version
- date
- subject
- other rule-specific fields

If fields are missing:
1. attempt extraction again,
2. use placeholders if still unavailable,
3. record missing fields for follow-up.

---

### Step 6.1 — Blur / Unreadable Page Handling
Blur or unreadable pages:
- must not stop the workflow,
- must be isolated as `Unclassified`,
- must be recorded in exception outputs,
- must not block later pages from being processed.

---

### Step 7 — Structured Output Package
Produce:
- classified PDF files
- file-level metadata
- processing report
- missing/unclassified report
- logs
- ZIP package

Recommended structure:

```text
TMF_Output.zip
├── Classified_Files/
├── Reports/
└── Logs/
```
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

This state model makes the workflow auditable and prevents overclaiming success on ambiguous, partial, or low-confidence results.

---

## Exception Handling

Exceptions are categorized rather than treated as generic failures.

### Identification Exceptions
- Document type cannot be determined
- Multiple plausible classes compete for the same content

### Boundary Exceptions
- Start or end page is unclear
- Same-type but multi-instance boundary is unstable

### Quality Exceptions
- Blur or unreadable pages
- Upside-down, rotated, or out-of-order pages

### Completeness Exceptions
- Partial EC submission chain
- Missing follow-up email or formal letter
- Incomplete IP receipt package
- Incomplete IP return package

### Output Exceptions
- Duplicate naming
- Missing metadata
- Page coverage is not closed

---

## Evaluation and Correctness

Correctness must be established through explicit validation criteria rather than subjective confidence alone.

### 1. Coverage Correctness
Check whether:

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

## Note on Examples and Tests
Real TMF example files and executable test fixtures are intentionally excluded from the public repository to avoid exposing confidential company information, project identifiers, and workflow-specific document structures.

The public repository includes the skills specification, processing framework, and validation criteria only. Real validation data should remain in an approved internal environment.
