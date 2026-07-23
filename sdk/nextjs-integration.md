---
title: "Next.js Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/nextjs-integration
canonical: https://vedika.io/docs/sdk/nextjs-integration
---

# Next.js Integration

Server Components and API Routes

## Setup

```
# Create Next.js app
npx create-next-app@latest my-astro-app --typescript --tailwind --app

# Install SDK
npm install @vedika-io/sdk
```

```
# .env.local
VEDIKA_API_KEY=vk_live_your_api_key
```

## API Route (App Router)

```
// app/api/chart/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    const chart = await vedika.birthChart({
      datetime: body.datetime,
      latitude: body.latitude,
      longitude: body.longitude
    });

    return NextResponse.json(chart);
  } catch (error: any) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Streaming API Route

```
// app/api/chat/stream/route.ts
import { NextRequest } from 'next/server';
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

export async function POST(request: NextRequest) {
  const { question, birthDetails } = await request.json();

  const stream = await vedika.queryStream({
    question,
    birthDetails
  });

  // Create a ReadableStream for SSE
  const encoder = new TextEncoder();
  const readable = new ReadableStream({
    async start(controller) {
      for await (const event of stream) {
        const data = JSON.stringify(event);
        controller.enqueue(encoder.encode(`data: ${data}\n\n`));
      }
      controller.close();
    }
  });

  return new Response(readable, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

## Server Component

```
// app/chart/[id]/page.tsx
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

interface PageProps {
  params: { id: string };
  searchParams: {
    datetime: string;
    lat: string;
    lng: string;
  };
}

export default async function ChartPage({ searchParams }: PageProps) {
  // Fetch data server-side
  const chart = await vedika.birthChart({
    datetime: searchParams.datetime,
    latitude: parseFloat(searchParams.lat),
    longitude: parseFloat(searchParams.lng)
  });

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">Your Birth Chart</h1>

      <div className="grid grid-cols-3 gap-4 mb-8">
        <div className="bg-yellow-50 p-4 rounded-lg text-center">
          <div className="text-sm text-gray-500">Sun Sign</div>
          <div className="text-2xl font-bold">{chart.sunSign}</div>
        </div>
        <div className="bg-blue-50 p-4 rounded-lg text-center">
          <div className="text-sm text-gray-500">Moon Sign</div>
          <div className="text-2xl font-bold">{chart.moonSign}</div>
        </div>
        <div className="bg-purple-50 p-4 rounded-lg text-center">
          <div className="text-sm text-gray-500">Ascendant</div>
          <div className="text-2xl font-bold">{chart.ascendant}</div>
        </div>
      </div>

      <h2 className="text-xl font-semibold mb-4">Planetary Positions</h2>
      <div className="bg-white shadow rounded-lg overflow-hidden">
        <table className="w-full">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-4 py-3 text-left">Planet</th>
              <th className="px-4 py-3 text-left">Sign</th>
              <th className="px-4 py-3 text-left">House</th>
            </tr>
          </thead>
          <tbody>
            {chart.planets.map((planet) => (
              <tr key={planet.name} className="border-t">
                <td className="px-4 py-3">{planet.name}</td>
                <td className="px-4 py-3">{planet.sign}</td>
                <td className="px-4 py-3">{planet.house}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

## Server Actions

```
// app/actions.ts
'use server';

import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY
});

export async function generateChart(formData: FormData) {
  const datetime = formData.get('datetime') as string;
  const latitude = parseFloat(formData.get('latitude') as string);
  const longitude = parseFloat(formData.get('longitude') as string);

  const chart = await vedika.birthChart({
    datetime,
    latitude,
    longitude
  });

  return chart;
}

export async function getPanchang(date: string, lat: number, lng: number) {
  return await vedika.panchang({ date, latitude: lat, longitude: lng });
}
```

```
// app/chart/form.tsx
'use client';

import { useState } from 'react';
import { generateChart } from '../actions';

export function ChartForm() {
  const [chart, setChart] = useState(null);

  async function handleSubmit(formData: FormData) {
    const result = await generateChart(formData);
    setChart(result);
  }

  return (
    <form action={handleSubmit}>
      <input name="datetime" type="datetime-local" required />
      <input name="latitude" type="number" step="0.0001" required />
      <input name="longitude" type="number" step="0.0001" required />
      <button type="submit">Generate Chart</button>
    </form>
  );
}
```

## Caching Strategies

```
// app/panchang/page.tsx
import { VedikaClient } from '@vedika-io/sdk';
import { unstable_cache } from 'next/cache';

const vedika = new VedikaClient();

// Cache panchang for 1 hour
const getCachedPanchang = unstable_cache(
  async (date: string, lat: number, lng: number) => {
    return vedika.panchang({ date, latitude: lat, longitude: lng });
  },
  ['panchang'],
  { revalidate: 3600 }  // 1 hour
);

export default async function PanchangPage() {
  const today = new Date().toISOString().split('T')[0];
  const panchang = await getCachedPanchang(today, 28.6139, 77.2090);

  return (
    <div>
      <h1>Today's Panchang</h1>
      <p>Tithi: {panchang.tithi.name}</p>
      <p>Nakshatra: {panchang.nakshatra.name}</p>
    </div>
  );
}
```

## Rate Limiting Middleware

```
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

const rateLimit = new Map<string, number[]>();
const WINDOW_MS = 60 * 1000;  // 1 minute
const MAX_REQUESTS = 10;

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/api/')) {
    const ip = request.ip || 'anonymous';
    const now = Date.now();

    const timestamps = rateLimit.get(ip) || [];
    const recent = timestamps.filter(t => t > now - WINDOW_MS);

    if (recent.length >= MAX_REQUESTS) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        { status: 429 }
      );
    }

    recent.push(now);
    rateLimit.set(ip, recent);
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/api/:path*'
};
```

## Next Steps

[ **

#### React Components

Reusable chart components

](https://vedika.io/docs/sdk/react-integration) [ **

#### Build Horoscope App

Complete Next.js app

](https://vedika.io/docs/sdk/horoscope-app)

---
_Full page (always current): [https://vedika.io/docs/sdk/nextjs-integration](https://vedika.io/docs/sdk/nextjs-integration) · [Vedika docs](https://vedika.io/docs)_
