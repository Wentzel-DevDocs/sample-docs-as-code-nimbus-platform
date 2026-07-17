# sample-nimbus-platform

The **complex** sample docs-as-code repo for the `hosted.devdocs.ai` multi-tenant docs product:
38 pages in a three-level nav across five product areas (Concepts, Guides, Reference, Tutorials,
Troubleshooting), dense internal cross-links, and a build-generated `llms.txt` site index. The
content documents **Nimbus**, a fictional cloud platform. This repo is the one for exercising
scale: chunking, retrieval under volume, and citation quality across a deep nav tree.

## Repo layout

```
content/
  index.md                       root page (path "index")
  concepts/                      6 pages — platform mental model
  guides/
    getting-started/             3 pages
    compute/                     4 pages
    data/                        3 pages
    observability/               3 pages
  reference/
    cli/                         4 pages
    api/                         4 pages
    config/                      2 pages
  tutorials/                     4 pages
  troubleshooting/               4 pages
scripts/publish.mjs              build + publish script (also generates llms.txt)
docs.config.json                 site title, default slug, generateLlmsTxt: true
.github/workflows/publish.yml    publishes from CI on push to main
```

Each folder's `_section.json` names and orders its nav group; page frontmatter (`title`,
`description`, `order`) orders pages within a group.

## One-time account setup (manual, done in the product)

1. Sign up at `https://hosted.devdocs.ai/signup` and verify the email.
2. Claim a subdomain in the dashboard (for example `nimbus`).
3. Create a **push token** in the dashboard — it starts with `hdpt_` and is shown once.

The token is scoped to the workspace that minted it, and the broker derives the storage path
from the token server-side, so this repo can only ever write its own site.

## Publish

```bash
npm install
PUSH_TOKEN=hdpt_xxx SLUG=<your-subdomain> node scripts/publish.mjs
# or: PUSH_TOKEN=hdpt_xxx SLUG=<your-subdomain> npm run publish
```

- `VERSION` is optional (env var or first CLI arg); it defaults to a timestamp.
- `SLUG` sets the live URL printed at the end **and** the absolute page URLs written into
  `llms.txt`, so set it (or `defaultSlug` in `docs.config.json`) to your claimed subdomain.
- A successful push (HTTP 200) **automatically becomes the live version**. The script prints the
  broker response (`versionId`, `entries`, `chunksIndexed`) and `https://<slug>.hosted.devdocs.ai`.

After publishing, open the site's chat widget and ask something the docs answer — for example,
"How do I add a custom domain to a service?" — to verify grounded citations under volume.

## Build without publishing (no token needed)

```bash
DRY_RUN=1 node scripts/publish.mjs
```

Writes the exact POST body to `dist/push-body.json` and unpacks every artifact under `dist/site/`
(including the generated `llms.txt`).

## Publish from CI (the intended model)

Add the push token as a repo secret named `HOSTED_DOCS_PUSH_TOKEN`; the included workflow builds
and publishes on every push to `main`, using the git SHA as the version.

## How the artifact format works

Each publish POSTs `{ manifest, files }` to `https://hosted.devdocs.ai/api/broker/push` with
`Authorization: Bearer <PUSH_TOKEN>`. `manifest.entries` lists every file with its lowercase hex
`sha256` and byte `size` (verified server-side); `files` maps the same paths to base64 bytes.
Artifacts per publish: one `manifest.json` (`{ version, siteTitle, nav }` — folders become
grouping NavEntries, files become leaves), a `<path>.page.json` per page (sanitized `bodyHtml`
with id anchors matching its `toc`), the verbatim `<path>.md` source that feeds the AI retrieval
index, and — for this repo — a generated `llms.txt` summarizing every page with its description
and markdown URL. The root page's path is exactly `index`.
