---
title: "Astrology API SDK: Python & JavaScript Examples | Vedika"
source: https://vedika.io/sdks
canonical: https://vedika.io/sdks
---

** Developer Resources

# SDKs & Code Examples

Copy-paste ready code snippets in Python, JavaScript, and cURL. All examples use the FREE Sandbox - run them immediately!

[ ** Python ](#python) [ ** JavaScript ](#javascript) [ ** cURL ](#curl)

## Quick Start (30 seconds)

FREE No API key needed for Sandbox
```
# Get a birth chart instantly (no signup required)
curl -X POST "https://api.vedika.io/sandbox/birth-chart" \
  -H "Content-Type: application/json" \
  -d '{"datetime":"1990-05-15T10:30:00","latitude":28.6139,"longitude":77.2090,"timezone":"+05:30"}'
```

## Python

### Installation Official SDK

```
# Install official Vedika SDK from PyPI
pip install vedika-sdk
```

[ **View on PyPI ](https://pypi.org/project/vedika-sdk/)

### Quick Start with SDK

```
from vedika_sdk import VedikaClient

# Initialize with your API key
client = VedikaClient(api_key="your_api_key")

# Or use free sandbox mode (no API key needed)
client = VedikaClient(sandbox=True)

# Ask astrology questions
response = client.ask(
    "What is my moon sign?",
    birth_details={
        "datetime": "1990-05-15T10:30:00+05:30",
        "latitude": 28.6139,
        "longitude": 77.2090
    }
)
print(response)
```

### Predictions NEW v1.1.0

```
from vedika_sdk import VedikaClient

client = VedikaClient(api_key="your_api_key")

# Daily prediction by rashi (moon sign)
daily = client.predict_daily(rashi="Aries")
print(f"Score: {daily['data']['overallScore']}")
print(f"Career: {daily['data']['predictions']['career']['text']}")

# Weekly, Monthly, Quarterly, Yearly predictions
weekly = client.predict_weekly(rashi="Taurus")
monthly = client.predict_monthly(rashi="Gemini")
quarterly = client.predict_quarterly(rashi="Leo")
yearly = client.predict_yearly(rashi="Scorpio")

# Or use birth details instead of rashi
prediction = client.predict_daily(birth_details={
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090
})
```

### Birth Chart (Kundali)

```
from vedika_sdk import VedikaClient

client = VedikaClient(api_key="your_api_key")

# Generate birth chart
chart = client.get_birth_chart(
    datetime="1990-05-15T10:30:00+05:30",
    latitude=28.6139,
    longitude=77.2090
)
print(f"Ascendant: {chart['data']['ascendant']['sign']}")
print(f"Moon Sign: {chart['data']['moon']['sign']}")
```

### Kundali Matching (36 Guna Milan)

```
import requests

def match_kundali(bride, groom):
    """Calculate compatibility between two birth charts"""
    response = requests.post(
        f"{SANDBOX_URL}/astrology/kundli-matching",
        json={
            "bride": bride,
            "groom": groom
        }
    )
    return response.json()

# Example
bride = {"date": "1995-03-15", "time": "10:30", "lat": 28.61, "lng": 77.20}
groom = {"date": "1992-07-22", "time": "14:15", "lat": 19.07, "lng": 72.87}

result = match_kundali(bride, groom)
print(f"Score: {result['total_score']}/36")
print(f"Verdict: {result['verdict']}")
```

### AI Chatbot Query

```
import requests

def ask_vedika(question, api_key, birth_details=None, language="en"):
    """Ask natural language astrology question (requires API key)"""
    payload = {
        "query": question,
        "language": language
    }
    if birth_details:
        payload["birth_details"] = birth_details

    response = requests.post(
        "https://api.vedika.io/api/v1/astrology/query",
        headers={"Authorization": f"Bearer {api_key}"},
        json=payload
    )
    return response.json()

# Example: General question
answer = ask_vedika("What does Mars in 7th house mean?", api_key="YOUR_API_KEY")
print(answer["response"])

# Example: Personalized question
birth = {"date": "1990-05-15", "time": "10:30", "lat": 28.61, "lng": 77.20}
answer = ask_vedika("How is my career looking this year?", api_key="YOUR_API_KEY", birth_details=birth, language="hi")
print(answer["response"])  # Response in Hindi
```

## JavaScript / Node.js

### Vedika CLI Official CLI

```
# Run directly with npx (no install needed)
npx @vedika-io/cli --help

# Or install globally
npm install -g @vedika-io/cli

# Login with your API key
vedika login

# Get predictions
vedika predict daily --rashi Aries
vedika predict weekly --rashi Taurus
vedika predict monthly --rashi Gemini

# Get panchang
vedika panchang
vedika panchang 2026-01-15

# AI astrology chat
vedika ask "What is my moon sign?" --datetime "1990-05-15T10:30:00" --lat 28.6139 --lon 77.2090
```

[ **View on npm ](https://www.npmjs.com/package/@vedika-io/cli)

### Browser (Fetch API)

```
const SANDBOX_URL = 'https://api.vedika.io/sandbox';

async function getKundali(datetime, latitude, longitude, timezone) {
  const response = await fetch(`${SANDBOX_URL}/birth-chart`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ datetime, latitude, longitude, timezone })
  });
  return response.json();
}

// Usage
const chart = await getKundali('1990-05-15T10:30:00', 28.6139, 77.2090, '+05:30');
console.log(`Ascendant: ${chart.data.ascendant.sign}`);
```

### Node.js (with axios)

```
const axios = require('axios');

const SANDBOX_URL = 'https://api.vedika.io/sandbox';
// For production:
// const PROD_URL = 'https://api.vedika.io';
// const headers = { 'Authorization': 'Bearer YOUR_API_KEY' };

async function matchKundali(bride, groom) {
  const { data } = await axios.post(
    `${SANDBOX_URL}/astrology/kundli-matching`,
    { bride, groom }
  );
  return data;
}

// Usage
const bride = { date: '1995-03-15', time: '10:30', lat: 28.61, lng: 77.20 };
const groom = { date: '1992-07-22', time: '14:15', lat: 19.07, lng: 72.87 };

const result = await matchKundali(bride, groom);
console.log(`Score: ${result.total_score}/36 - ${result.verdict}`);
```

### React Hook Example

```
import { useState, useEffect } from 'react';

function useHoroscope(sign, period = 'daily') {
  const [horoscope, setHoroscope] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchHoroscope() {
      try {
        const res = await fetch(
          `https://api.vedika.io/sandbox/horoscope/${period}`
        );
        const data = await res.json();
        setHoroscope(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    fetchHoroscope();
  }, [sign, period]);

  return { horoscope, loading, error };
}

// Usage in component
function DailyHoroscope({ sign }) {
  const { horoscope, loading } = useHoroscope(sign);
  if (loading) return <p>Loading...</p>;
  return <p>{horoscope.prediction}</p>;
}
```

## cURL

### Birth Chart

```
curl -X POST "https://api.vedika.io/sandbox/birth-chart" \
  -H "Content-Type: application/json" \
  -d '{"datetime":"1990-05-15T10:30:00","latitude":28.6139,"longitude":77.2090,"timezone":"+05:30"}'
```

### Kundali Matching

```
curl -X POST "https://api.vedika.io/sandbox/astrology/kundli-matching" \
  -H "Content-Type: application/json" \
  -d '{
    "bride": {"date": "1995-03-15", "time": "10:30", "lat": 28.61, "lng": 77.20},
    "groom": {"date": "1992-07-22", "time": "14:15", "lat": 19.07, "lng": 72.87}
  }'
```

### AI Chatbot Query

```
curl -X POST "https://api.vedika.io/api/v1/astrology/query" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "query": "What does Saturn in 10th house mean for career?",
    "language": "en"
  }'
```

### Daily Panchang

```
curl "https://api.vedika.io/sandbox/panchang/today?latitude=28.6139&longitude=77.2090&timezone=%2B05%3A30"
```

### Production API (with API Key)

```
# Replace YOUR_API_KEY with your actual API key from dashboard
curl -X POST "https://api.vedika.io/v2/astrology/kundli" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"datetime":"1990-05-15T10:30:00","latitude":28.6139,"longitude":77.2090,"timezone":"+05:30"}'
```

## Ready to Build?

Start with the FREE Sandbox - no signup required. When you're ready for production, get your API key from the dashboard.

[ Read Full Docs ](docs) [ Get API Key ](dashboard)

---
_Full page (always current): [https://vedika.io/sdks](https://vedika.io/sdks) · [Vedika docs](https://vedika.io/docs)_
