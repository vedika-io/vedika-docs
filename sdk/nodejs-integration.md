---
title: "Node.js Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/nodejs-integration
canonical: https://vedika.io/docs/sdk/nodejs-integration
---

# Node.js Integration

Express, Fastify, and serverless

## Express.js Setup

```
// server.js
import express from 'express';
import { VedikaClient } from '@vedika-io/sdk';

const app = express();
app.use(express.json());

const vedika = new VedikaClient();

app.post('/api/chart', async (req, res) => {
  try {
    const chart = await vedika.birthChart(req.body);
    res.json({
      sunSign: chart.sunSign,
      moonSign: chart.moonSign,
      ascendant: chart.ascendant,
      planets: chart.planets
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/api/match', async (req, res) => {
  const { bride, groom } = req.body;
  const match = await vedika.kundliMatch({ bride, groom });
  res.json({
    score: match.totalScore,
    verdict: match.verdict
  });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

## Fastify Setup

```
// server.js
import Fastify from 'fastify';
import { VedikaClient } from '@vedika-io/sdk';

const fastify = Fastify({ logger: true });
const vedika = new VedikaClient();

fastify.post('/api/chart', {
  schema: {
    body: {
      type: 'object',
      required: ['datetime', 'latitude', 'longitude'],
      properties: {
        datetime: { type: 'string' },
        latitude: { type: 'number' },
        longitude: { type: 'number' }
      }
    }
  }
}, async (request, reply) => {
  const chart = await vedika.birthChart(request.body);
  return { sunSign: chart.sunSign, moonSign: chart.moonSign };
});

fastify.listen({ port: 3000 });
```

## Streaming Response

```
// Express SSE streaming
app.post('/api/chat/stream', async (req, res) => {
  const { question, birthDetails } = req.body;

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    const stream = await vedika.queryStream({ question, birthDetails });

    for await (const event of stream) {
      res.write(`data: ${JSON.stringify(event)}\n\n`);
    }

    res.end();
  } catch (error) {
    res.write(`data: ${JSON.stringify({ type: 'error', error: error.message })}\n\n`);
    res.end();
  }
});
```

## AWS Lambda

```
// handler.js
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

export const handler = async (event) => {
  const body = JSON.parse(event.body);

  try {
    const chart = await vedika.birthChart(body);

    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        sunSign: chart.sunSign,
        moonSign: chart.moonSign,
        ascendant: chart.ascendant
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};
```

## Vercel Edge Functions

```
// api/chart.js
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const chart = await vedika.birthChart(req.body);
  res.json(chart);
}

export const config = {
  runtime: 'edge',
};
```

## Rate Limiting Middleware

```
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 20, // 20 requests per minute
  message: { error: 'Too many requests, please try again later' }
});

app.use('/api/', apiLimiter);
```

## Redis Caching

```
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

async function getCachedOrFetch(key, ttl, fetchFn) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const data = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify(data));
  return data;
}

app.get('/api/panchang', async (req, res) => {
  const { date, lat, lng } = req.query;
  const key = `panchang:${date}:${lat}:${lng}`;

  const panchang = await getCachedOrFetch(key, 3600, () =>
    vedika.panchang({ date, latitude: parseFloat(lat), longitude: parseFloat(lng) })
  );

  res.json(panchang);
});
```

## Next Steps

[ **

#### Next.js

Full-stack framework

](https://vedika.io/docs/sdk/nextjs-integration) [ **

#### Streaming API

Real-time responses

](https://vedika.io/docs/sdk/streaming)

---
_Full page (always current): [https://vedika.io/docs/sdk/nodejs-integration](https://vedika.io/docs/sdk/nodejs-integration) · [Vedika docs](https://vedika.io/docs)_
