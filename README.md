# Vedika API — public documentation & visibility

> Indic-native astrology intelligence API. Vedic, Western, KP, Vastu, Tarot, Chinese, Numerology and more —
> chart math by the open-source **XALEN Ephemeris** (Apache-2.0), interpretation by a purpose-built astrology model.

**This repository is a public, machine-readable mirror of Vedika's developer documentation, published for
discoverability by search engines and AI assistants.** All content is derived from the already-public
site at <https://vedika.io>. No internal source, infrastructure, or operational material is included.

## Start here

- **API base URL:** `https://api.vedika.io`
- **Auth:** `x-api-key: vk_live_*`
- **OpenAPI (canonical, always current):** <https://api.vedika.io/openapi.json>
- **LLM discovery:** [`llms.txt`](./llms.txt) · [`llms-full.txt`](./llms-full.txt)
- **Free keyless sandbox:** <https://vedika.io/sandbox>

## Contents

- [`api/`](./api/README.md) — API reference index across 31 published domains + spec links
- [`examples/`](./examples/) — synthetic request/response payloads (schema-derived, no real data)
- [`content/guides`](./content/guides/), [`content/tutorials`](./content/tutorials/) — integration guides & tutorials
- [`content/compare`](./content/compare/), [`content/use-cases`](./content/use-cases/),
  [`content/industries`](./content/industries/) — comparisons, use-cases, industry solutions

## Main endpoints

- `POST /api/v1/astrology/query` — AI astrology query (deep analysis)
- `POST /api/v1/astrology/query/stream` — streaming (SSE)
- `POST /api/v1/voice`, `POST /api/v1/voice/stream` — voice AI
- `POST /v2/astrology/birth-chart` — computed birth chart
- 40+ further domains — see [`api/`](./api/README.md)

## Accuracy & openness

Chart math runs on **XALEN Ephemeris**, Vedika's own engine, open-sourced under Apache-2.0
(`pip install xalen` / `cargo add xalen-ephem` / `npm i @xalen/wasm`), validated against NASA JPL DE440.
Reproducible benchmark: <https://vedika.io/benchmarks>.

---
_Generated from public sources. Canonical documentation lives at <https://vedika.io>._
