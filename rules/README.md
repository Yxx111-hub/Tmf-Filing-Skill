# Rules Folder

This folder contains configurable rule templates for TMF Filing Skill.

## Rule File 1: TMF_Naming_Rules_Template.xlsx

This file defines document types and naming conventions.

Recommended columns:

```text
Classification No.
Document Type
Document Subtype
Document Classification
Expected Document
Uploading Keywords
Description Text
Requiredness
Expected Count
Document Example
Short Name Requirement
Version Name
```
## Rule File 2: PDF_Special_Processing_Rules_Template.xlsx
This file defines special processing rules.
It is used to support:

- Content-based splitting
- Split and merge rules
- Special document handling
- Completeness checks
- Exception detection

Recommended columns:
```text
- Rule_ID
- Rule_Name
- Rule_Type
- Document_Type
- Trigger_Condition
- Splitting_Logic
- Merging_Logic
- Output_Format
- Validation_Check
- Priority
- Notes
```
For completeness rules, recommended columns:

```text
- Rule_ID
- Rule_Name
- Rule_Type
- Document_Group
- Trigger_Condition
- Required_Components
- Conditional_Components
- Optional_Components
- Grouping_Key
- Action
- Output_Format
- Validation_Check
- Missing_Message
- Priority
- Notes
```
## Relationship Between Rule Files
The two rule files work together:

- TMF_Naming_Rules_Template.xlsx defines what a document is and how it should be named.
- PDF_Special_Processing_Rules_Template.xlsx defines how complex PDF content should be split, merged, grouped, or validated.

The skill should first identify document types using the TMF naming rules, then apply special processing rules when needed.
## Rule Types
### Splitting
Used when a PDF should be split into multiple independent files.
Examples
```text
Combined PDF → CV + GCP Certificate + Medical License
```
### Merge
Used when multiple documents should be combined into one file.
Examples
```text
Email + Follow-up Letter → One Follow-up Letter PDF
```
### Split+Merge
Used when parts must first be identified separately and then reconstructed into one final document.
Examples
```text
Sponsor to Investigator Letter
+
Investigator to Ethics Committee Letter
=
EC Submission Letter
```

### Completeness
Used when a group of related documents must be checked for required components.
Examples
```text
IMP/IP Shipment Receipt Package
```
The rule should define:

- Required components
- Conditional components
- Optional components
- Missing item message
- Validation logic
## How to Use These Rule Files

1. Review TMF_Naming_Rules_Template.xlsx to confirm document types and naming conventions.
2. Review PDF_Special_Processing_Rules_Template.xlsx to confirm special splitting, merging, and completeness rules.
3. Customize the rule files according to your own study, SOP, or document process.
4. Use the customized rule files together with PDF inputs when running the TMF Filing Skill.


## Public Repository Reminder
Do not upload confidential or company-specific rule files.

Use sanitized templates only.

Do not include:

- Company-specific SOP text
- Internal project names
- Real study numbers
- Real site information
- Real employee names
- Real patient information
- Internal system URLs
