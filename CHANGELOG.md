## V2.0

### Added
- PDF preprocessing for watermark and fixed promotional header/footer suppression during analysis
- orientation and reading-order normalization as a mandatory gate
- hard constraints for full page coverage, no early stop, deduplication, and secondary segmentation
- explicit state management
- explicit exception categories
- correctness evaluation framework
- boundary-case handling and failure-mode definition
- file-level metadata and output validation rules

### Improved
- content-level segmentation instead of page-based splitting
- special-rule handling for EC submission, Follow-up Letter, IP Receipt, and IP Return
- auditability of output files through metadata, reports, and logs

### Validation
- coverage correctness
- pipeline correctness
- type correctness
- segmentation correctness
- special-rule correctness
- naming correctness
- auditability

### Public Repository Note
- Real examples and runnable TMF test fixtures are intentionally excluded from the public repository due to confidentiality and project-data exposure concerns.
- Public examples and tests are replaced by documentation-based placeholders and validation scope descriptions.
