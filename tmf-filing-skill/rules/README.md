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
### Recommended columns:
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
### For completeness rules, recommended columns:
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
## Rule Types
### Splitting
Used when a PDF should be split into multiple independent files.
### Merge
Used when multiple documents should be combined into one file.
### Split+Merge
Used when parts must first be identified separately and then reconstructed into one final document.
### Completeness
Used when a group of related documents must be checked for required components.
## Public Repository Reminder
Do not upload confidential or company-specific rule files.
Use sanitized templates only.
