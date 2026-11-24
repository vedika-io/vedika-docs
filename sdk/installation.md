---
title: "JavaScript SDK Installation | Vedika API Documentation"
source: https://vedika.io/docs/sdk/installation
canonical: https://vedika.io/docs/sdk/installation
---

# JavaScript SDK Installation

@vedika-io/sdk

**v3.3.0  Last updated: November 2026

## Quick Install

Install the Vedika SDK using your preferred package manager:

npm
```
npm install @vedika-io/sdk
```

yarn
```
yarn add @vedika-io/sdk
```

pnpm
```
pnpm add @vedika-io/sdk
```

## Requirements

| Node.js | >= 18.0.0 |
| --- | --- |
| TypeScript | >= 4.7 (optional, types included) |
| Browser Support | ES2020+ (Chrome 80+, Firefox 74+, Safari 14+) |

## Verify Installation

After installation, verify the SDK is working:
```
import { VedikaClient } from '@vedika-io/sdk';

// Check version
console.log(VedikaClient.VERSION);

// Initialize client
const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

// Test connection
const health = await vedika.health();
console.log('API Status:', health.status); // 'ok'
```

## TypeScript Setup

The SDK includes TypeScript definitions out of the box. No additional setup required.
```
import {
  VedikaClient,
  BirthChart,
  KundliMatch,
  Panchang,
  VedikaError
} from '@vedika-io/sdk';

// Full type safety
const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY!,
  timeout: 30000,
  retries: 3
});

// Response types are fully typed
const chart: BirthChart = await vedika.birthChart({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});
```

**Recommended tsconfig.json settings:** `"moduleResolution": "bundler"` or `"node16"`

## Browser Usage

The package is built for bundlers and Node (`npm install @vedika-io/sdk`); it does not ship a prebuilt browser script, so there is no script-tag or CDN install. For a browser app, import it through your bundler.

**Security Warning:** Never expose your API key in client-side code for production apps. Use a backend proxy instead.

## Environment Variables

The SDK automatically reads from environment variables:
```
# .env file
VEDIKA_API_KEY=YOUR_API_KEY
VEDIKA_BASE_URL=https://api.vedika.io  # Optional, defaults to production
VEDIKA_TIMEOUT=30000                    # Optional, milliseconds
VEDIKA_MAX_RETRIES=3                    # Optional
```

```
// The SDK auto-detects VEDIKA_API_KEY
import { VedikaClient } from '@vedika-io/sdk';

// No need to pass apiKey if env var is set
const vedika = new VedikaClient();
```

## Package Contents

#### Main Exports

- `VedikaClient` - Main client class
- `VedikaError` - Error types
- `createClient` - Factory function

#### Type Exports

- `BirthChart` - Chart response type
- `KundliMatch` - Match response type
- `Panchang` - Panchang type
- `DashaPeriod` - Dasha type

## Next Steps

[

#### Quickstart Guide

Your first API call in 5 minutes

](https://vedika.io/docs/sdk/quickstart) [

#### Authentication

API key setup and best practices

](https://vedika.io/docs/sdk/authentication)

---
_Full page (always current): [https://vedika.io/docs/sdk/installation](https://vedika.io/docs/sdk/installation) · [Vedika docs](https://vedika.io/docs)_
