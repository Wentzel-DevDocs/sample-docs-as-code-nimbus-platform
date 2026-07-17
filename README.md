# sample-nimbus-platform

**Complex** enterprise docs-as-code sample for [hosted.devdocs.ai](https://hosted.devdocs.ai).

Documents **Nimbus**, a fictional cloud platform, with a three-level nav, Diátaxis sections, glossary, release notes, redirect map, Mermaid architecture diagrams, and generated `llms.txt`. Use this when you want volume: chunking, deep navigation, and cross-page retrieval.

| Scale | Nav | Pages | Extra artifacts |
| --- | --- | ---: | --- |
| Complex | Three-level | 40 | `llms.txt`, `redirects.json` |

## Why this looks like an enterprise platform docs repo

- **Concepts / Guides / Tutorials / Reference / Troubleshooting** information architecture
- Nested nav groups via `_section.json` at every folder level
- Glossary + release notes as first-class pages
- Sample `config/redirects.json` for path migrations (published artifact)
- Mermaid diagrams in concept pages
- Enterprise page metadata on every file
- Full CI: quality gate on PRs, build + optional publish on `main`
- `CODEOWNERS`, contribution guide, and style guide for multi-team ownership

## Repo layout

```
content/
  index.md
  release-notes.md
  concepts/                + glossary
  guides/
    getting-started/
    compute/
    data/
    observability/
  reference/
    cli/
    api/
    config/
  tutorials/
  troubleshooting/
config/redirects.json      sample migration map (published)
docs.config.json
scripts/check.mjs
scripts/publish.mjs
.github/
CONTRIBUTING.md
STYLEGUIDE.md
```

## Content map (40 pages)

| Area | Count | Notes |
| --- | ---: | --- |
| Root | 2 | Welcome, release notes |
| Concepts | 7 | Includes glossary |
| Guides | 13 | Getting started, compute, data, observability |
| Reference | 10 | CLI, REST API, config |
| Tutorials | 4 | End-to-end builds |
| Troubleshooting | 4 | Failures + FAQ |

## Local workflow

```bash
npm ci
npm test
PUSH_TOKEN=hdpt_xxx SLUG=nimbus npm run publish   # optional
```

Dry-run output includes `dist/site/llms.txt` and `dist/site/redirects.json`.

## Structural rules (enterprise)

1. Every nav folder has `_section.json` with `title` + `order`.
2. Every page has `title`, `description`, `order`, plus recommended ownership metadata.
3. Internal links use root-absolute paths (`/guides/compute/scaling`).
4. Renames update `config/redirects.json` in the same PR.

## One-time product setup

1. Sign up at `https://hosted.devdocs.ai/signup`.
2. Claim a subdomain (for example `nimbus`).
3. Create a push token; set `HOSTED_DOCS_PUSH_TOKEN` for CI publish.

`SLUG` / `defaultSlug` also drives absolute URLs inside generated `llms.txt`.

## Verify after publish

Ask: **How do I add a custom domain to a service?**

Also try a cross-section question such as: **What is the difference between a project and an environment?**

## Related samples

| Repo | Intent |
| --- | --- |
| [hello-docs](https://github.com/Wentzel-DevDocs/sample-docs-as-code-hello-docs) | Simple / flat |
| [acme-api](https://github.com/Wentzel-DevDocs/sample-docs-as-code-acme-api) | Medium / API + OpenAPI |
| [nimbus-platform](https://github.com/Wentzel-DevDocs/sample-docs-as-code-nimbus-platform) | Complex / platform |
