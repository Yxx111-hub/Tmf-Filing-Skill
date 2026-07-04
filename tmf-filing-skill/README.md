# TMF Filing Skill

A configurable TMF PDF filing skill for content-based PDF splitting, TMF naming, special document handling, and structured output generation.

## Overview

TMF Filing Skill helps users process combined PDF files by:

- Identifying document content
- Matching document types against TMF naming rules
- Splitting PDFs based on content, not page count
- Applying special handling rules for complex documents
- Renaming files according to TMF naming conventions
- Generating a structured TMF output package
- Creating processing and exception reports

This repository is designed as a generic, reusable template. It does not contain company-specific information.

---

## Core Principle

The skill must not split PDFs page by page.

Instead, it should analyze the content and determine logical document boundaries.

A document may be:

- 1 page
- 10+ pages
- 20+ pages
- Part of a larger combined PDF

Splitting should happen only when the document type, content purpose, submission direction, or logical document boundary changes.

---

## Knowledge Sources

The skill uses two configurable rule files:

### 1. TMF Naming Rules

Used to define:

- Document type
- Document subtype
- Document classification
- Uploading keywords
- Short name requirements
- Version name requirements
- Naming format

Most document types should be matched from this rule file.

### 2. PDF Special Processing Rules

Used to define complex processing logic, including:

- Content-based splitting
- Split and merge rules
- Submission letter handling
- Ethics approval handling
- Follow-up letter handling
- IMP/IP receipt completeness checks

---

## Recommended Repository Structure

```text
tmf-filing-skill/
├── README.md
├── skill.md
├── rules/
│   ├── TMF_Naming_Rules_Template.xlsx
│   ├── PDF_Special_Processing_Rules_Template.xlsx
│   └── README.md
├── examples/
│   ├── sample_processing_report.md
│   └── sample_folder_structure.md
├── docs/
│   └── rule_design_guide.md
└── LICENSE
``` 
## Processing Workflow
Input PDF
   ↓
Read TMF naming rules
   ↓
Read PDF special processing rules
   ↓
Analyze PDF content
   ↓
Identify document type
   ↓
Determine logical document boundaries
   ↓
Apply special split / merge / completeness rules
   ↓
Rename according to TMF naming rules
   ↓
Generate structured folders
   ↓
Create TMF output zip package
   ↓
Generate processing report
## Key Features
### Content-Based Splitting
The skill splits PDFs based on document content, not by fixed page count.
Examples:

- A 20-page CV should remain one CV document.
- A combined PDF containing a CV, GCP certificate, and medical license should be split into separate files.
- A submission package may need to be logically segmented and then reconstructed into one final TMF document.
### TMF Naming
Output files are named using the configured TMF naming rules.
If required naming fields are missing, the skill should flag the item in the exception report.
### Special Processing Rules
The skill supports special processing scenarios such as:

- EC submission letters
- EC SAE submissions
- EC approval letters
- Follow-up letters
- IMP/IP shipment receipt packages

## Example Special Rules

### EC Submission Letter

If the document contains both:

- Sponsor to Investigator communication
- Investigator to Ethics Committee communication

Then the skill should:

1. Identify both parts.
2. Keep Sponsor to Investigator first.
3. Keep Investigator to Ethics Committee second.
4. Merge both parts into one EC Submission Letter PDF.
5. Flag missing parts in the report.

---

### EC SAE Submission

If the document contains Investigator to Ethics Committee communication and mentions SAE or serious adverse event, the skill should treat it as an EC SAE submission.

Sponsor to Investigator communication is not required for this rule.

---

### EC Approval Letter

If the document is an EC Approval Letter and the review type is meeting review, the skill should identify:

- The approval letter
- The ethics committee composition list from the same meeting date

The final output should place:

1. EC Approval Letter first
2. Ethics committee composition list second

---

### Follow-up Letter

If a Follow-up Letter is detected, the skill should identify:

- The email
- The follow-up letter

The email should be placed before the follow-up letter.

The visit date should be consistent between both parts.

---

### IMP / IP Shipment Receipt

For IMP/IP shipment receipt packages, the skill should group documents by:

- Shipment
- Batch
- Receipt date

Default required components may include:

- Courier receipt
- Drug shipment document
- Randomization or IWRS/IRT receipt confirmation
- Temperature record during shipment

Optional or configurable components may include:

- Receipt photo
- Email communication
- Warehouse entry record
- Temperature deviation report
- Temperature logger calibration certificate
- Shipment request form
- Drug return receipt
- Drug return details

Users should customize required, conditional, and optional components based on their process.

---

## Output

The skill should generate:

```text
TMF_Output.zip
├── Classified_Files/
│   ├── Trial_Management/
│   ├── Site_Management/
│   ├── Ethics/
│   ├── IMP_IP/
│   └── Other/
├── Reports/
│   ├── Processing_Report.xlsx
│   └── Missing_or_Unclassified_Items.xlsx
└── Logs/
    └── Processing_Log.txt
```
### Exception Handling
The skill should flag:

1. Unclassified content
2. Missing required components
3. Unclear document boundaries
4. Missing naming fields
5. Duplicate file names
6. Incomplete split-and-merge document groups
7. Potential mismatch between related documents
## Configuration
Users can adapt this skill by editing the rule files:
1. rules/TMF_Naming_Rules_Template.xlsx
2. rules/PDF_Special_Processing_Rules_Template.xlsx
3. No company-specific information should be stored in this public repository.
## Intended Users
This template may be useful for:

- Clinical Research Associates
- TMF specialists
- Clinical document management teams
- Quality control reviewers
- Clinical operations teams
- AI agent builders working on document automation
## Disclaimer
This repository provides a configurable framework for TMF document processing.
Users are responsible for validating outputs according to their organization’s SOPs, study requirements, and applicable regulations.