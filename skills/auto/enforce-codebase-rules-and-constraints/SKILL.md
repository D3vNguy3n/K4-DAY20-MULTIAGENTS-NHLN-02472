---
name: enforce-codebase-rules-and-constraints
description: When modifying code repositories or outputting structured files, use this checklist to comply with all formatting, testing, and metadata rules.
---
- Verify and obey all explicit codebase rules before finalizing work:
  - Do not modify restricted files or test directories unless explicitly permitted.
  - Add required regression tests (e.g., one test function per bug fix) in dedicated new test files.
  - Update required changelogs under the correct headings with appropriate bullet formats.
  - Add strict type annotations to all parameters and return values of public functions.
- Format structured outputs (JSON/CSV) exactly as specified:
  - Ensure monetary values are stored in integer cents rather than floats when required.
  - Include all mandatory metadata objects (e.g., `meta` keys matching input row counts and source names).
  - Normalize keys, identifiers, and categorical values according to specification (e.g., service name casing/replacements).
- Run the full test suite and inspect outputs for compliance before submission.
