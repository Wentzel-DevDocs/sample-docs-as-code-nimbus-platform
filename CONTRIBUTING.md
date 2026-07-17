# Contributing to Nimbus platform docs

This is the large-sample enterprise docs-as-code repository: multi-level navigation, Diátaxis sections, glossary, release notes, redirect map, and `llms.txt` generation.

## Local loop

```bash
npm ci
npm test   # check + dry-run build (includes llms.txt + redirects.json)
```

## Content architecture (Diátaxis-inspired)

| Section | Intent |
| --- | --- |
| Concepts | Understanding — mental models |
| Guides | Doing — task-oriented how-tos |
| Tutorials | Learning — end-to-end lessons |
| Reference | Looking up — CLI, API, config |
| Troubleshooting | Fixing — failures and FAQs |

## Structural rules

- Every folder that appears in nav has `_section.json` with `title` and `order`.
- Pages declare `order` within their folder; keep orders unique.
- Internal links use root-absolute paths (`/guides/compute/scaling`).
- When you rename or remove a page, add a redirect in `config/redirects.json`.

## Ownership

`CODEOWNERS` routes reference paths to API docs owners and everything else to the docs writers team. Adjust for your org.
