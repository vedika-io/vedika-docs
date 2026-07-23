# Push checklist (founder-gated)

Staged locally by the build tooling. Nothing pushed; no GitHub repo exists yet.

1. **Pick a name** (yours to decide) — e.g. `vedika-docs`, `vedika-api-docs`.
2. **Review `LEAK-SCAN-REPORT.md`** — verdict PASS (tripwire=0, secrets=0, masking=0).
3. **Skim `MANIFEST.md`** — every file traces to a published `vedika.io` URL, `llms.txt`,
   or synthetic construction. Nothing from an internal tree.
3b. **Content provenance:** prose sourced from local files whose URLs are published;
    NOT byte-verified against live except two spot-checked pages (LEAK-SCAN §2).
    Skim pages vs their `canonical:` URL before publishing.
3c. **SDK package names were registry-corrected** (LEAK-SCAN §8): `@vedika/sdk`→`@vedika-io/sdk`,
    `vedika-python`→`vedika-sdk`. Confirm `@vedika-io/cli` and version pins/extras before push.
4. **License: All Rights Reserved** (decided) — see `NOTICE.md`. No OSI LICENSE file.
5. **Spot-check volatile claims** in `llms.txt`/`llms-full.txt` (pricing, availability) vs live.
6. **API spec** — repo links to canonical live `https://api.vedika.io/openapi.json`; do NOT
   commit the local 557-op superset (has admin headers).
7. **Create + push yourself:**
   ```
   cd vedika-public-staging
   git init && git add -A && git commit -m "Vedika public API docs"
   gh repo create vedika-io/<name> --public --source=. --push
   ```

## Phase 2 (deferred)
- 15 thin/JS-rendered pages dropped (re-source from live).
- 3,600+ granular `/docs/api/*` per-endpoint pages not inlined (bots covered by
  `llms-full.txt` + canonical spec link).
- `sdks/` SOURCE trees intentionally excluded (internal; not in sitemap).
