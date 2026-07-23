---
title: "Build an Astrology SaaS | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/astrology-saas
canonical: https://vedika.io/docs/sdk/astrology-saas
---

# Build an Astrology SaaS

Multi-tenant platform with white-label support

## Overview

Build a complete multi-tenant astrology SaaS platform that allows your customers to offer astrology services under their own brand. This guide covers authentication, billing, usage tracking, and white-label customization.

#### Multi-Tenant

Isolated customer data

#### White-Label

Custom branding

#### Usage Billing

Pay per API call

## Architecture

```

┌─────────────────────────────────────────────────────────────┐
│                     Your SaaS Platform                       │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ Tenant A │  │ Tenant B │  │ Tenant C │  │ Tenant D │    │
│  │ (Brand X)│  │ (Brand Y)│  │ (Brand Z)│  │ (Brand W)│    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
│       │             │             │             │           │
│       └─────────────┴──────┬──────┴─────────────┘           │
│                            │                                 │
│                    ┌───────▼───────┐                        │
│                    │  API Gateway  │                        │
│                    │  (Your Code)  │                        │
│                    └───────┬───────┘                        │
└────────────────────────────┼────────────────────────────────┘
                             │
                     ┌───────▼───────┐
                     │  Vedika API   │
                     │  (Upstream)   │
                     └───────────────┘

```

## Database Schema

```
-- Tenants (your customers)
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(63) UNIQUE,
    custom_domain VARCHAR(255) UNIQUE,
    api_key VARCHAR(64) UNIQUE NOT NULL,

    -- Branding
    logo_url TEXT,
    primary_color VARCHAR(7) DEFAULT '#6366f1',
    company_name VARCHAR(255),

    -- Billing
    plan VARCHAR(20) DEFAULT 'starter',
    credits_balance DECIMAL(12,2) DEFAULT 0,
    stripe_customer_id VARCHAR(255),

    -- Limits
    monthly_api_limit INTEGER DEFAULT 10000,
    rate_limit_per_minute INTEGER DEFAULT 60,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Tenant Users
CREATE TABLE tenant_users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) DEFAULT 'user',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, email)
);

-- API Usage Tracking
CREATE TABLE api_usage (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    endpoint VARCHAR(255) NOT NULL,
    credits_used DECIMAL(8,4) NOT NULL,
    response_time_ms INTEGER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create indexes
CREATE INDEX idx_api_usage_tenant_date ON api_usage(tenant_id, created_at);
CREATE INDEX idx_tenants_subdomain ON tenants(subdomain);
CREATE INDEX idx_tenants_custom_domain ON tenants(custom_domain);
```

## Tenant Resolution Middleware

```
// middleware/tenant.ts
import { Request, Response, NextFunction } from 'express';
import { db } from '../db';

interface Tenant {
  id: string;
  name: string;
  apiKey: string;
  plan: string;
  creditsBalance: number;
  rateLimitPerMinute: number;
  branding: {
    logoUrl: string;
    primaryColor: string;
    companyName: string;
  };
}

declare global {
  namespace Express {
    interface Request {
      tenant?: Tenant;
    }
  }
}

export async function resolveTenant(req: Request, res: Response, next: NextFunction) {
  // Try API key first (for API requests)
  const apiKey = req.headers['x-api-key'] as string;
  if (apiKey) {
    const tenant = await db.query(
      'SELECT * FROM tenants WHERE api_key = $1',
      [apiKey]
    );
    if (tenant.rows[0]) {
      req.tenant = formatTenant(tenant.rows[0]);
      return next();
    }
  }

  // Try subdomain
  const host = req.hostname;
  const subdomain = host.split('.')[0];

  if (subdomain && subdomain !== 'www' && subdomain !== 'api') {
    const tenant = await db.query(
      'SELECT * FROM tenants WHERE subdomain = $1 OR custom_domain = $2',
      [subdomain, host]
    );
    if (tenant.rows[0]) {
      req.tenant = formatTenant(tenant.rows[0]);
      return next();
    }
  }

  res.status(404).json({ error: 'Tenant not found' });
}

function formatTenant(row: any): Tenant {
  return {
    id: row.id,
    name: row.name,
    apiKey: row.api_key,
    plan: row.plan,
    creditsBalance: parseFloat(row.credits_balance),
    rateLimitPerMinute: row.rate_limit_per_minute,
    branding: {
      logoUrl: row.logo_url,
      primaryColor: row.primary_color,
      companyName: row.company_name
    }
  };
}
```

## Vedika API Proxy

```
// services/vedikaProxy.ts
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY // Your master API key
});

// Pricing per endpoint (in credits)
const ENDPOINT_PRICING: Record = {
  'birth-chart': 0.10,
  'kundli-match': 0.15,
  'panchang': 0.05,
  'dasha': 0.08,
  'dosha': 0.08,
  'daily-horoscope': 0.02,
  'query': 0.25 // AI queries
};

export async function proxyToVedika(
  tenantId: string,
  endpoint: string,
  params: any
): Promise<{ data: any; creditsUsed: number }> {
  const startTime = Date.now();

  // Check tenant credits
  const tenant = await db.query(
    'SELECT credits_balance FROM tenants WHERE id = $1',
    [tenantId]
  );

  const creditsNeeded = ENDPOINT_PRICING[endpoint] || 0.10;

  if (tenant.rows[0].credits_balance < creditsNeeded) {
    throw new Error('Insufficient credits');
  }

  // Call Vedika API
  let data: any;
  switch (endpoint) {
    case 'birth-chart':
      data = await vedika.birthChart(params);
      break;
    case 'kundli-match':
      data = await vedika.kundliMatch(params);
      break;
    case 'panchang':
      data = await vedika.panchang(params);
      break;
    case 'dasha':
      data = await vedika.dasha(params);
      break;
    case 'dosha':
      data = await vedika.dosha(params);
      break;
    default:
      throw new Error(`Unknown endpoint: ${endpoint}`);
  }

  const responseTime = Date.now() - startTime;

  // Deduct credits and log usage
  await db.query(`
    UPDATE tenants SET credits_balance = credits_balance - $1 WHERE id = $2;
    INSERT INTO api_usage (tenant_id, endpoint, credits_used, response_time_ms)
    VALUES ($2, $3, $1, $4);
  `, [creditsNeeded, tenantId, endpoint, responseTime]);

  return { data, creditsUsed: creditsNeeded };
}
```

## Per-Tenant Rate Limiting

```
// middleware/rateLimiter.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function tenantRateLimiter(req: Request, res: Response, next: NextFunction) {
  if (!req.tenant) return next();

  const key = `ratelimit:${req.tenant.id}`;
  const limit = req.tenant.rateLimitPerMinute;
  const window = 60; // 1 minute

  const current = await redis.incr(key);

  if (current === 1) {
    await redis.expire(key, window);
  }

  // Set rate limit headers
  res.setHeader('X-RateLimit-Limit', limit);
  res.setHeader('X-RateLimit-Remaining', Math.max(0, limit - current));

  if (current > limit) {
    const ttl = await redis.ttl(key);
    res.setHeader('X-RateLimit-Reset', Math.floor(Date.now() / 1000) + ttl);

    return res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: ttl
    });
  }

  next();
}
```

## Stripe Billing Integration

```
// routes/billing.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Credit packages
const CREDIT_PACKAGES = [
  { id: 'credits_1000', amount: 1000, price: 999 },   // $9.99
  { id: 'credits_5000', amount: 5000, price: 3999 },  // $39.99
  { id: 'credits_20000', amount: 20000, price: 14999 } // $149.99
];

router.post('/purchase-credits', async (req, res) => {
  const { packageId } = req.body;
  const pkg = CREDIT_PACKAGES.find(p => p.id === packageId);

  if (!pkg) {
    return res.status(400).json({ error: 'Invalid package' });
  }

  const session = await stripe.checkout.sessions.create({
    customer: req.tenant.stripeCustomerId,
    payment_method_types: ['card'],
    line_items: [{
      price_data: {
        currency: 'usd',
        product_data: {
          name: `${pkg.amount} API Credits`,
          description: 'Vedika API credits for astrology endpoints'
        },
        unit_amount: pkg.price
      },
      quantity: 1
    }],
    mode: 'payment',
    success_url: `${process.env.APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/billing/cancelled`,
    metadata: {
      tenant_id: req.tenant.id,
      credits: pkg.amount.toString()
    }
  });

  res.json({ checkoutUrl: session.url });
});

// Webhook handler
router.post('/webhook', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(
    req.body,
    sig,
    process.env.STRIPE_WEBHOOK_SECRET
  );

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object;
    const { tenant_id, credits } = session.metadata;

    await db.query(
      'UPDATE tenants SET credits_balance = credits_balance + $1 WHERE id = $2',
      [parseInt(credits), tenant_id]
    );
  }

  res.json({ received: true });
});
```

## White-Label Configuration

```
// routes/branding.ts
router.get('/branding', async (req, res) => {
  res.json({
    logoUrl: req.tenant.branding.logoUrl || '/default-logo.png',
    primaryColor: req.tenant.branding.primaryColor,
    companyName: req.tenant.branding.companyName || req.tenant.name,
    features: getPlanFeatures(req.tenant.plan)
  });
});

router.put('/branding', async (req, res) => {
  const { logoUrl, primaryColor, companyName } = req.body;

  await db.query(`
    UPDATE tenants
    SET logo_url = $1, primary_color = $2, company_name = $3, updated_at = NOW()
    WHERE id = $4
  `, [logoUrl, primaryColor, companyName, req.tenant.id]);

  res.json({ success: true });
});

function getPlanFeatures(plan: string) {
  const features = {
    starter: {
      endpoints: ['birth-chart', 'panchang', 'daily-horoscope'],
      customDomain: false,
      whiteLabel: false,
      support: 'email'
    },
    professional: {
      endpoints: ['birth-chart', 'panchang', 'dasha', 'dosha', 'kundli-match', 'daily-horoscope'],
      customDomain: true,
      whiteLabel: true,
      support: 'priority'
    },
    enterprise: {
      endpoints: 'all',
      customDomain: true,
      whiteLabel: true,
      support: 'dedicated'
    }
  };
  return features[plan] || features.starter;
}
```

## Usage Analytics Dashboard

```
// routes/analytics.ts
router.get('/usage', async (req, res) => {
  const { startDate, endDate } = req.query;

  // Daily usage breakdown
  const dailyUsage = await db.query(`
    SELECT
      DATE(created_at) as date,
      endpoint,
      COUNT(*) as calls,
      SUM(credits_used) as credits
    FROM api_usage
    WHERE tenant_id = $1
      AND created_at >= $2
      AND created_at <= $3
    GROUP BY DATE(created_at), endpoint
    ORDER BY date DESC
  `, [req.tenant.id, startDate, endDate]);

  // Summary stats
  const summary = await db.query(`
    SELECT
      COUNT(*) as total_calls,
      SUM(credits_used) as total_credits,
      AVG(response_time_ms) as avg_response_time
    FROM api_usage
    WHERE tenant_id = $1
      AND created_at >= $2
      AND created_at <= $3
  `, [req.tenant.id, startDate, endDate]);

  res.json({
    summary: summary.rows[0],
    dailyBreakdown: dailyUsage.rows,
    currentBalance: req.tenant.creditsBalance
  });
});
```

## Dynamic Theme (React)

```
// contexts/TenantContext.tsx
import { createContext, useContext, useEffect, useState } from 'react';

interface TenantBranding {
  logoUrl: string;
  primaryColor: string;
  companyName: string;
}

const TenantContext = createContext<TenantBranding | null>(null);

export function TenantProvider({ children }) {
  const [branding, setBranding] = useState<TenantBranding | null>(null);

  useEffect(() => {
    fetch('/api/branding')
      .then(res => res.json())
      .then(data => {
        setBranding(data);
        // Apply theme
        document.documentElement.style.setProperty('--primary-color', data.primaryColor);
      });
  }, []);

  if (!branding) return <div>Loading...</div>;

  return (
    <TenantContext.Provider value={branding}>
      {children}
    </TenantContext.Provider>
  );
}

export function useTenant() {
  return useContext(TenantContext);
}

// Usage in components
function Header() {
  const tenant = useTenant();

  return (
    <header className="bg-primary">
      <img src={tenant.logoUrl} alt={tenant.companyName} />
      <h1>{tenant.companyName}</h1>
    </header>
  );
}
```

## Deployment Checklist

**Database**

PostgreSQL with connection pooling (PgBouncer/Neon)

**Redis**

For rate limiting and session storage

**Wildcard SSL**

*.yourdomain.com for subdomain tenants

**CDN**

Cloudflare for custom domains and caching

## Next Steps

[ **

#### Horoscope App

Daily predictions

](https://vedika.io/docs/sdk/horoscope-app) [ **

#### Matching Portal

Matrimonial sites

](https://vedika.io/docs/sdk/matching-portal)

---
_Full page (always current): [https://vedika.io/docs/sdk/astrology-saas](https://vedika.io/docs/sdk/astrology-saas) · [Vedika docs](https://vedika.io/docs)_
