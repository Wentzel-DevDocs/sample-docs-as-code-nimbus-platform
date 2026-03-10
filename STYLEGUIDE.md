# Style guide (Nimbus sample)

## Diátaxis

- **Tutorials** are learning-oriented and opinionated ("build this app").
- **Guides** are task-oriented ("add a custom domain").
- **Reference** is exhaustive and dry (flags, fields, endpoints).
- **Concepts** explain systems without step lists.

Do not mix a full tutorial and a full reference table on the same page.

## Diagrams

Mermaid is welcome for architecture and sequence flows. Keep diagrams under ~15 nodes so they stay readable in the rendered docs and in source review.

## Release communication

User-visible platform changes need:

1. A [release notes](/release-notes) bullet.
2. Updates to the affected reference/guide page in the **same PR**.
3. A redirect if a path moved.
