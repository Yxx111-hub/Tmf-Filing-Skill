# Tests

This public repository does not include real executable TMF test fixtures.

## Why real test fixtures are excluded
Real TMF test files may reveal:
- confidential company information
- project-specific structures
- document naming conventions tied to active studies
- shipment, tracking, or logger identifiers
- workflow details not suitable for public exposure

## Test scope preserved in documentation
The workflow is still designed to cover the following validation areas:

- page coverage correctness
- preprocessing correctness
- orientation normalization correctness
- deduplication
- secondary segmentation
- special-rule grouping
- naming correctness
- auditability
- boundary-case handling
- failure-mode detection

## Recommended internal validation strategy
For internal validation, use:
- synthetic samples
- anonymized or redacted PDFs
- privately maintained fixtures stored outside the public repository