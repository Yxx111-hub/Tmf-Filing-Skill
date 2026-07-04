# Rule Design Guide

## 1. Design Principle

Rules should be configurable and generic.

The skill should contain the processing engine.
The Excel rule files should contain organization-specific or study-specific details.

## 2. Recommended Rule Structure

Each rule should answer:

```text
When should this rule be triggered?
What document type or document group does it apply to?
Should the document be split?
Should parts be merged?
Should required components be checked?
What should the final output be?
What validation should be performed?
What message should be shown if something is missing?
```
## 3. Required / Conditional / Optional Model
For completeness checks, use three levels:
### Required
Must exist. If missing, the skill should generate a warning.
### Conditional
Required only when a specific condition applies. If unclear, the skill should ask the user to confirm.
### Optional
Included if detected. Missing optional files should not generate warnings.
## 4. Example: IMP/IP Shipment Receipt
### Required Components
Courier receipt
Drug shipment document
Randomization or IWRS/IRT receipt confirmation
Temperature record during shipment
### Conditional Components
Temperature deviation report
Temperature logger calibration certificate
Cold-chain handover record
### Optional Components
Receipt photo
Email communication
Warehouse entry record
Supplemental notes
Drug return receipt
Drug return detail sheet
## 5. Example: EC Submission Letter
### Trigger:
Sponsor to Investigator communication
+
Investigator to Ethics Committee communication
### Action:
Split logically.
Place Sponsor to Investigator first.
Place Investigator to Ethics Committee second.
Merge into one EC Submission Letter.
### Validation:
Both parts should exist.
If one part is missing, flag the issue.
## 6. Avoid Hardcoding
Do not hardcode company-specific requirements into the skill.
Put configurable details in rule files.


