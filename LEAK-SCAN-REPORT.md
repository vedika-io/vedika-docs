# Leak-scan report

**Verdict: PASS**  —  files scanned: 80  ·  deny-tripwire: 0  ·  secrets: 0  ·  masking-flags: 0

Full-tree scan (every staged file). Staged locally; nothing pushed.

## 1. Provenance model (primary defense)
- **Allowlist = the live sitemap.** A file ships only if its `vedika.io` URL appears in the published sitemaps. Admin/auth/dashboard surfaces are absent from the sitemap and were never pulled.
- **Deny-tripwire** hard-checks that no staged path traces to an internal tree (`docs/ops`, `docs/private`, `docs/repo-map`, `server/`, `rust/`, `infra/`, `memory/`, `war-room/`, `.claude/.agents/.codex`, admin/auth). Result below.

## 2. Content provenance — HONEST CAVEAT
Prose pages were sourced from **local files whose URLs are published**, then converted to markdown. They are **NOT byte-verified against the live site**, EXCEPT the spot-checks below. The local `web/vedika-public/` tree is known to be a *superset* in at least one place (its `openapi.json` had 557 ops vs ~115 served live) — so a local prose page *could* carry a draft/unreleased section not on the live page. Mitigation + evidence:
- Spot-checked `compare/vedika-vs-prokerala` — live headings and all pricing ($12/$60/$120/$240; $19/$49/$99) **match local exactly**.
- Spot-checked `use-cases/kundli-matching-app` — live content headings **match local** (local correctly dropped the site footer nav).
- **Before push:** skim each shipped page against its `canonical:` URL, or re-source from the live URL, to fully rule out local-only draft content.

## 3. Deny-tripwire (must be 0)
0 hits.
- none

## 4. Secret scan (must be 0)
0 live-secret matches outside example context.
- none

## 5. Masking review (human-decide, non-blocking)
0 tokens matched infra/admin patterns.
- none

_Note: the astrological term **vertex** (a chart point) appears in `llms-full.txt` and is NOT an infra reference (not GCP Vertex AI) — excluded from the masking pattern as a known false positive._

## 6. Spec handling
The local `openapi.json` (557 ops, contains `x-admin-key`/`x-admin-secret` headers, superset of the ~115-op live spec) is **NOT re-hosted**. The repo links to the canonical live spec at `https://api.vedika.io/openapi.json`. Do not commit the local spec copy without confirming the exact public op-set.

## 7. Thin pages removed from v1
0 near-empty JS-rendered shells (mostly tutorials) were dropped rather than ship thin/doorway content that could hurt SEO. Re-source these from their live URLs in a phase 2:
- none

## 8. SDK docs — package-claim verification (registry-checked)
SDK documentation was added under `sdk/`. Every install command was checked against the live npm / PyPI / pub.dev registries; **verifiably-false package names were corrected** (58 edits across 25 files). The local `sdks/` SOURCE trees were NOT published (internal; absent from sitemap).

**Verified real:**
- `@vedika-io/sdk` — npm v3.0.5 ✓
- `vedika-sdk` — PyPI v3.0.5 ✓, imports as `vedika_sdk`
- `@vedika-io/mcp-server` — npm v2.0.3 ✓
- `@vedika-io/cli` — npm v1.1.0 ✓

**Corrected (were false → now real):**
- `@vedika/sdk` → `@vedika-io/sdk` (npm) — npm 404 for @vedika/sdk; @vedika-io/sdk exists v3.0.5
- `vedika-python` → `vedika-sdk` (PyPI) — PyPI 404 for vedika-python; vedika-sdk exists v3.0.5

**Unverified / confirm before push:**
- vedika_sdk (Flutter/pub.dev) 404 NOT FOUND matches llms.txt; no Flutter install is present in staged docs (vedika_sdk here is the Python import name, which is correct)
- version pins like vedika-sdk==1.8.0 predate real latest 3.0.5; and pip extras [async]/[cli]/[all] unverified — founder confirm or drop pins
