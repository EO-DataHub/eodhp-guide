---
title: Definition of Done
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
# Definition of Done

## Source Code Updated

- Source code implementation is completed (issue is solved).
- No TODOs in the source code.
- Code is maintainable.
- Refactor to remove unnecessary duplication.
- No security vulnerabilities.
- Appropriate error handling and logging.
- All commented out code removed.
- Code linted for dead code.
- No build errors.
- Code formatted according to coding standards.
- CI/CD pipelines updated so that deployment of the changes is fully automated.

## Tested

- New features or fixes are unit tested.
- Unit tests pass.
- Regression tests pass.

## Integrated

- Code should be tested in the development environment, where possible.
- Integration tests pass.

## Documented

- README.md updated.
- CHANGELOG.md updated.
- Any further documentation updated, e.g. docs/ or user manual.
- Updates are adequate, correct and in line with the change.

## Reviewed

- Pull request created and linked to the Jira issue.
- Any important design decisions documented in PR summary.
- Peer reviewed by at least one required reviewer:
  - Code complete and correctly implement the design.
  - Commented adequately and correctly.
  - Well formatted and structured.
  - Readable.
  - Sufficient and clear documentation.
  - Appropriate tests.

## Code Merged

- Merged to appropriate branch (usually main or develop).
- Any merge conflicts are resolved.
- Commit tagged with appropriate version, if required.
