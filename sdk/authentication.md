---
title: "SDK Authentication | Vedika API Documentation"
source: https://vedika.io/docs/sdk/authentication
canonical: https://vedika.io/docs/sdk/authentication
---

# Authentication

Secure API key management

## API Key Types

TEST `vk_test_*`

For development and testing. Returns mock data. Free to use.

LIVE `vk_live_*`

For production. Real calculations. Pay-per-use billing.

## Basic Authentication

```
import { VedikaClient } from '@vedika-io/sdk';

// Option 1: Pass API key directly
const vedika = new VedikaClient({
  apiKey: 'YOUR_API_KEY'
});

// Option 2: Use environment variable (recommended)
// The SDK auto-reads VEDIKA_API_KEY from process.env
const vedika = new VedikaClient();

// Option 3: Custom environment variable name
const vedika = new VedikaClient({
  apiKey: process.env.MY_CUSTOM_KEY_NAME
});
```

## Environment Variables Setup

Create a `.env` file in your project root:
```
# .env
VEDIKA_API_KEY=YOUR_API_KEY

# Optional configuration
VEDIKA_BASE_URL=https://api.vedika.io  # Default
VEDIKA_TIMEOUT=30000                    # 30 seconds
VEDIKA_MAX_RETRIES=3                    # Retry failed requests
```

**Important:** Add `.env` to your `.gitignore` file to prevent accidentally committing your API key.

## Framework-Specific Setup

### Node.js with dotenv

```
// Install: npm install dotenv
import 'dotenv/config';
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient(); // Reads VEDIKA_API_KEY automatically
```

### Next.js

```
// .env.local
VEDIKA_API_KEY=YOUR_API_KEY

// pages/api/astrology.js or app/api/astrology/route.ts
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

export async function POST(request) {
  const { birthDetails } = await request.json();
  const chart = await vedika.birthChart(birthDetails);
  return Response.json(chart);
}
```

### Vite / React (Server-Side Only)

```
// vite.config.js - expose env vars
export default defineConfig({
  define: {
    'process.env.VEDIKA_API_KEY': JSON.stringify(process.env.VEDIKA_API_KEY)
  }
});

// IMPORTANT: Only use in server-side code (API routes, SSR)
// Never expose API keys in client-side React components
```

## Security Best Practices

#### DO: Use environment variables

Store API keys in environment variables, not in code.

#### DO: Keep keys server-side

Never expose API keys in client-side JavaScript. Use backend proxies.

#### DO: Rotate keys regularly

Regenerate keys periodically and after any suspected compromise.

#### DON'T: Commit keys to git

Add .env to .gitignore immediately.

#### DON'T: Share keys publicly

Never share API keys in forums, GitHub issues, or support tickets.

## Backend Proxy Pattern

For frontend apps, create a backend API route to proxy requests:
```
// Backend: /api/astrology/chart.js
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient(); // Reads from env

export async function handler(req, res) {
  const { datetime, latitude, longitude } = req.body;

  try {
    const chart = await vedika.birthChart({
      datetime,
      latitude,
      longitude
    });
    res.json(chart);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch chart' });
  }
}

// Frontend: React component
async function fetchChart(birthDetails) {
  const response = await fetch('/api/astrology/chart', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(birthDetails)
  });
  return response.json();
}
```

## Key Management

Manage your API keys in the [Vedika Dashboard](https://vedika.io/dashboard):
- Create:** Generate new API keys
- Rotate:** Regenerate existing keys
- Revoke:** Delete compromised keys
- Monitor:** Track usage per key

## Testing Authentication

```
import { VedikaClient } from '@vedika-io/sdk';

async function testAuth() {
  const vedika = new VedikaClient();

  try {
    // Test connection with health check
    const health = await vedika.health();
    console.log('Connected!', health.status);

    // Verify API key with usage check
    const usage = await vedika.usage();
    console.log('API Key valid. Credits remaining:', usage.creditsRemaining);
  } catch (error) {
    if (error.code === 'INVALID_API_KEY') {
      console.error('Invalid API key');
    } else if (error.code === 'EXPIRED_API_KEY') {
      console.error('API key has expired');
    } else {
      console.error('Connection failed:', error.message);
    }
  }
}

testAuth();
```

## Next Steps

[ **

#### Birth Chart API

Generate horoscopes

](https://vedika.io/docs/sdk/birth-chart) [ **

#### Error Handling

Handle auth errors gracefully

](https://vedika.io/docs/sdk/error-handling)

---
_Full page (always current): [https://vedika.io/docs/sdk/authentication](https://vedika.io/docs/sdk/authentication) · [Vedika docs](https://vedika.io/docs)_
