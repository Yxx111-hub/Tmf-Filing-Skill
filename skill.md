
# Skill Definition: TMF Filing PDF Split & Naming

## Skill Name

TMF Filing PDF Split & Naming Skill

---

## Purpose

This skill processes uploaded PDF files for TMF filing by identifying document content, splitting the PDF based on document type, applying special handling rules, renaming files according to TMF naming rules, and generating a structured output package.

---

## Inputs

The skill accepts:

1. One or more PDF files.
2. A TMF naming rule file.
3. A PDF special processing rule file.

---

## Outputs

The skill generates:

1. Split and renamed PDF files.
2. A structured TMF output folder.
3. A compressed output package.
4. A processing report.
5. A missing or exception report.

---

## Required Rule Files

### TMF Naming Rules

Used for:

- Document type identification
- Document subtype identification
- Naming convention
- Short name rules
- Version rules
- Classification logic

### PDF Special Processing Rules

Used for:

- Content-based splitting rules
- Split-and-merge rules
- Completeness checks
- Special document handling
- Exception detection

---

## General Processing Logic

1. Load TMF naming rules.
2. Load PDF special processing rules.
3. Analyze PDF content.
4. Identify document types.
5. Determine logical split boundaries.
6. Apply special handling rules.
7. Rename files according to TMF naming rules.
8. Organize files into structured folders.
9. Generate output zip package.
10. Generate processing and exception reports.

---

## Mandatory Rule: Content-Based Splitting

The skill must never split a PDF simply by page.

The skill must split based on:

- Document title
- Document type
- Submission direction
- Content topic
- Signature section
- Date consistency
- Attachment relationship
- Logical document boundary

A single document may contain one page or many pages.

---

## Special Rule Types

### Splitting
```text
Used when one PDF contains multiple independent document types.

Example:

Combined PDF → CV + GCP Certificate + Medical License

Output 3 separate pdf 
1.CV.pdf
2.GCP Certificate.pdf
3.Medical License.pdf
```
### Merge
Used when separate documents need to be combined into one final TMF document.

### Split+Merge
```text
Used when a PDF contains several logical parts that must be identified separately and reconstructed into one final document.
Example:
Sponsor to Investigator letter
+
Investigator to Ethics Committee letter
=
EC Submission Letter
```
## Special Processing Examples
### EC Submission Letter
```text
Trigger condition:
Sponsor to Investigator section
+
Investigator to Ethics Committee section
Action:
Identify both parts.
Place Sponsor to Investigator first.
Place Investigator to Ethics Committee second.
Merge into one EC Submission Letter.
Validation:
Both parts must exist.
If one part is missing, flag the issue.
```
### EC SAE Submission
```text
Trigger condition:
Investigator to Ethics Committee section
+
SAE or serious adverse event content
Action
Classify as EC SAE Submission.
Do not require Sponsor to Investigator section.
Output as one PDF.
```
### EC Approval Letter
```text
Trigger condition:
EC Approval Letter
+
Meeting review
Action:
Identify EC Approval Letter.
Identify ethics committee composition list from the same meeting date.
Place approval letter first.
Place composition list second.
Merge into one PDF.
Validation:
Meeting date should be consistent.
If composition list is missing, flag for review.
```
### Follow-up Letter
```text
Trigger condition:
Follow-up letter or visit follow-up letter detected
Action：
Identify email section.
Identify follow-up letter section.
Place email first.
Place follow-up letter second.
Merge into one PDF.
Validation:
Visit date in email and follow-up letter should match.
```
### IMP/IP Shipment Receipt
```text
Trigger condition:
Drug shipment, receipt, IWRS/IRT, or temperature record content detected
Action:
Group related documents by shipment ID, batch, or receipt date.
Check required components.
Include optional components if detected.
Generate completeness report.
Default required components:
Courier receipt
Drug shipment document
Randomization or IWRS/IRT receipt confirmation
Temperature record during shipment
Optional or configurable components:
Receipt photo
Email communication
Warehouse entry record
Temperature deviation report
Temperature logger calibration certificate
Shipment request form
Drug return receipt
Drug return details
Validation:
Required missing = warning
Optional missing = no warning
Conditional missing = ask user to confirm applicability
```
## Naming Logic
Each output PDF must be named according to the TMF naming rule file.
If required naming information is missing:

1. Use the best available extracted value.
2. Add a placeholder if necessary.
3. Record the issue in the exception report.
## Output Package
### The final output should be:
TMF_Output.zip
### Suggested structure:
TMF_Output/
├── Classified_Files/
├── Reports/
└── Logs/
## Validation Checklist
The skill should verify:
- All pages are processed.
- No pages are lost.
- No page-by-page incorrect splitting occurs.
- File types are matched against TMF naming rules.
- Special handling rules are applied.
- Required components are checked.
- Output files are named correctly.
- Duplicate names are avoided.
- Exceptions are reported.
## Privacy and Public Use
This skill is designed as a generic template.
### Do not include:

- Company names
- Internal project names
- Real study numbers
- Real site information
- Real patient information
- Real employee names
- Internal system URLs
- Confidential SOP text

Users should provide their own sanitized rule files.
