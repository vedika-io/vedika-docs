---
title: "React Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/react-integration
canonical: https://vedika.io/docs/sdk/react-integration
---

# React Integration

Build astrology components with React

## Setup

```
# Create React app with TypeScript
npx create-react-app my-astro-app --template typescript

# Install SDK
npm install @vedika-io/sdk
```

**Security Note:** Never expose your API key in client-side React code. Always use a backend proxy or Next.js API routes.

## Backend Proxy Pattern

```
// server/api.js (Express backend)
import express from 'express';
import { VedikaClient } from '@vedika-io/sdk';

const app = express();
const vedika = new VedikaClient(); // Reads from env

app.post('/api/chart', async (req, res) => {
  try {
    const chart = await vedika.birthChart(req.body);
    res.json(chart);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// React fetches from your backend, not Vedika directly
```

## Custom Hook: useVedika

```
// hooks/useVedika.ts
import { useState, useCallback } from 'react';

interface BirthDetails {
  datetime: string;
  latitude: number;
  longitude: number;
}

interface BirthChart {
  sunSign: string;
  moonSign: string;
  ascendant: string;
  planets: any[];
}

export function useVedika() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchChart = useCallback(async (birthDetails: BirthDetails): Promise<BirthChart | null> => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/chart', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(birthDetails)
      });

      if (!response.ok) {
        throw new Error('Failed to fetch chart');
      }

      return await response.json();
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
      return null;
    } finally {
      setLoading(false);
    }
  }, []);

  return { fetchChart, loading, error };
}
```

## Birth Chart Component

```
// components/BirthChartForm.tsx
import { useState } from 'react';
import { useVedika } from '../hooks/useVedika';

interface BirthChartFormProps {
  onChartGenerated: (chart: any) => void;
}

export function BirthChartForm({ onChartGenerated }: BirthChartFormProps) {
  const [date, setDate] = useState('');
  const [time, setTime] = useState('');
  const [lat, setLat] = useState('');
  const [lng, setLng] = useState('');

  const { fetchChart, loading, error } = useVedika();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const chart = await fetchChart({
      datetime: `${date}T${time}:00+05:30`,
      latitude: parseFloat(lat),
      longitude: parseFloat(lng)
    });

    if (chart) {
      onChartGenerated(chart);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium">Date of Birth</label>
        <input
          type="date"
          value={date}
          onChange={(e) => setDate(e.target.value)}
          className="mt-1 block w-full rounded-md border-gray-300"
          required
        />
      </div>

      <div>
        <label className="block text-sm font-medium">Time of Birth</label>
        <input
          type="time"
          value={time}
          onChange={(e) => setTime(e.target.value)}
          className="mt-1 block w-full rounded-md border-gray-300"
          required
        />
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium">Latitude</label>
          <input
            type="number"
            step="0.0001"
            value={lat}
            onChange={(e) => setLat(e.target.value)}
            placeholder="28.6139"
            className="mt-1 block w-full rounded-md border-gray-300"
            required
          />
        </div>
        <div>
          <label className="block text-sm font-medium">Longitude</label>
          <input
            type="number"
            step="0.0001"
            value={lng}
            onChange={(e) => setLng(e.target.value)}
            placeholder="77.2090"
            className="mt-1 block w-full rounded-md border-gray-300"
            required
          />
        </div>
      </div>

      {error && (
        <div className="text-red-600 text-sm">{error}</div>
      )}

      <button
        type="submit"
        disabled={loading}
        className="w-full bg-indigo-600 text-white py-2 px-4 rounded-md hover:bg-indigo-700 disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Chart'}
      </button>
    </form>
  );
}
```

## Chart Display Component

```
// components/ChartDisplay.tsx
interface ChartDisplayProps {
  chart: {
    sunSign: string;
    moonSign: string;
    ascendant: string;
    planets: Array<{
      name: string;
      sign: string;
      degree: number;
      house: number;
    }>;
  };
}

export function ChartDisplay({ chart }: ChartDisplayProps) {
  return (
    <div className="bg-white rounded-lg shadow p-6">
      <h2 className="text-2xl font-bold mb-4">Your Birth Chart</h2>

      <div className="grid grid-cols-3 gap-4 mb-6">
        <div className="text-center p-4 bg-yellow-50 rounded-lg">
          <div className="text-sm text-gray-500">Sun Sign</div>
          <div className="text-xl font-bold">{chart.sunSign}</div>
        </div>
        <div className="text-center p-4 bg-blue-50 rounded-lg">
          <div className="text-sm text-gray-500">Moon Sign</div>
          <div className="text-xl font-bold">{chart.moonSign}</div>
        </div>
        <div className="text-center p-4 bg-purple-50 rounded-lg">
          <div className="text-sm text-gray-500">Ascendant</div>
          <div className="text-xl font-bold">{chart.ascendant}</div>
        </div>
      </div>

      <h3 className="text-lg font-semibold mb-3">Planetary Positions</h3>
      <table className="w-full">
        <thead>
          <tr className="border-b">
            <th className="text-left py-2">Planet</th>
            <th className="text-left py-2">Sign</th>
            <th className="text-left py-2">Degree</th>
            <th className="text-left py-2">House</th>
          </tr>
        </thead>
        <tbody>
          {chart.planets.map((planet) => (
            <tr key={planet.name} className="border-b">
              <td className="py-2">{planet.name}</td>
              <td className="py-2">{planet.sign}</td>
              <td className="py-2">{planet.degree.toFixed(2)}</td>
              <td className="py-2">{planet.house}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## Streaming Chat Component

```
// components/AstrologyChat.tsx
import { useState, useRef, useEffect } from 'react';

interface Message {
  role: 'user' | 'assistant';
  content: string;
}

interface AstrologyChatProps {
  birthDetails: {
    datetime: string;
    latitude: number;
    longitude: number;
  };
}

export function AstrologyChat({ birthDetails }: AstrologyChatProps) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const [streaming, setStreaming] = useState(false);
  const [currentResponse, setCurrentResponse] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(scrollToBottom, [messages, currentResponse]);

  const sendMessage = async () => {
    if (!input.trim() || streaming) return;

    const userMessage = input.trim();
    setInput('');
    setMessages(prev => [...prev, { role: 'user', content: userMessage }]);
    setStreaming(true);
    setCurrentResponse('');

    try {
      const response = await fetch('/api/chat/stream', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          question: userMessage,
          birthDetails
        })
      });

      const reader = response.body?.getReader();
      const decoder = new TextDecoder();

      let fullResponse = '';

      while (reader) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');

        for (const line of lines) {
          if (line.startsWith('data: ')) {
            try {
              const data = JSON.parse(line.slice(6));
              if (data.type === 'text') {
                fullResponse += data.text;
                setCurrentResponse(fullResponse);
              }
            } catch {}
          }
        }
      }

      setMessages(prev => [...prev, { role: 'assistant', content: fullResponse }]);
    } catch (error) {
      setMessages(prev => [...prev, {
        role: 'assistant',
        content: 'Sorry, I encountered an error. Please try again.'
      }]);
    } finally {
      setStreaming(false);
      setCurrentResponse('');
    }
  };

  return (
    <div className="flex flex-col h-96 border rounded-lg">
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((msg, i) => (
          <div
            key={i}
            className={`p-3 rounded-lg ${
              msg.role === 'user'
                ? 'bg-indigo-100 ml-auto max-w-[80%]'
                : 'bg-gray-100 mr-auto max-w-[80%]'
            }`}
          >
            {msg.content}
          </div>
        ))}
        {streaming && currentResponse && (
          <div className="bg-gray-100 mr-auto max-w-[80%] p-3 rounded-lg">
            {currentResponse}
            <span className="animate-pulse">|</span>
          </div>
        )}
        <div ref={messagesEndRef} />
      </div>

      <div className="border-t p-4">
        <div className="flex gap-2">
          <input
            type="text"
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
            placeholder="Ask about your chart..."
            className="flex-1 border rounded-lg px-4 py-2"
            disabled={streaming}
          />
          <button
            onClick={sendMessage}
            disabled={streaming || !input.trim()}
            className="bg-indigo-600 text-white px-4 py-2 rounded-lg disabled:opacity-50"
          >
            Send
          </button>
        </div>
      </div>
    </div>
  );
}
```

## Next Steps

[ **

#### Next.js Integration

API routes and SSR

](https://vedika.io/docs/sdk/nextjs-integration) [ **

#### Build Horoscope App

Complete React app guide

](https://vedika.io/docs/sdk/horoscope-app)

---
_Full page (always current): [https://vedika.io/docs/sdk/react-integration](https://vedika.io/docs/sdk/react-integration) · [Vedika docs](https://vedika.io/docs)_
