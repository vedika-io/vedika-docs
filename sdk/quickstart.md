---
title: "Astrology API Quickstart | AI Chatbot Integration | Kundali API Tutorial | Vedika"
source: https://vedika.io/quickstart
canonical: https://vedika.io/quickstart
---

** 5 Minutes ** FREE Sandbox

# Quickstart Guide

Get your first Vedika API call working in under 5 minutes. Instant sandbox access. No signup needed for sandbox.

Progress Step 0 of 4

[  Run in Postman ](https://www.postman.com/vedikaai/workspace/intelligence-platform/collection/50475884-27902efa-15c1-4d70-8dd3-72e874cc9be0) [ ** API Explorer ](https://vedika.io/api-explorer) [ ** Full Docs ](https://vedika.io/docs)

1

## Get Your Sandbox URL

The sandbox environment returns realistic mock data - perfect for development. **No API key required!**

Sandbox Base URL

`https://api.vedika.io/sandbox`

Tip:** When you're ready for production with real astronomical data, get your API key from the [Console](https://vedika.io/console).

2

## Make Your First Request

Let's get a daily horoscope. Choose your preferred language:
```
import requests

# Sandbox URL - no API key needed!
url = "https://api.vedika.io/sandbox/horoscope/daily"

params = {
    "sign": "aries",
    "language": "en"
}

response = requests.get(url, params=params)
data = response.json()

print(f"Today's Horoscope for Aries:")
print(data["horoscope"])
print(f"\nLucky Number: {data['lucky_number']}")
print(f"Lucky Color: {data['lucky_color']}")
```

```
// Using fetch (Node 18+) or install node-fetch
const url = "https://api.vedika.io/sandbox/horoscope/daily";

const params = new URLSearchParams({
    sign: "aries",
    language: "en"
});

fetch(`${url}?${params}`)
    .then(res => res.json())
    .then(data => {
        console.log("Today's Horoscope for Aries:");
        console.log(data.horoscope);
        console.log(`\nLucky Number: ${data.lucky_number}`);
        console.log(`Lucky Color: ${data.lucky_color}`);
    })
    .catch(err => console.error(err));
```

```
# Simple GET request - no API key needed for sandbox!
curl -X GET "https://api.vedika.io/sandbox/horoscope/daily?sign=aries&language=en"

# Pretty print with jq (if installed)
curl -s "https://api.vedika.io/sandbox/horoscope/daily?sign=aries&language=en" | jq
```

3

## Parse the Response

Here's what the JSON response looks like:
```
{
  "success": true,
  "sign": "aries",
  "date": "2025-12-26",
  "horoscope": "Today brings exciting opportunities for career growth. Your natural leadership abilities will shine in team settings. Financial matters look favorable - consider reviewing investments.",
  "lucky_number": 7,
  "lucky_color": "Red",
  "lucky_time": "2:00 PM - 4:00 PM",
  "mood": "Energetic",
  "compatibility": "Leo",
  "element": "Fire",
  "ruling_planet": "Mars"
}
```

#### Response Fields

- `horoscope` - Daily prediction text
- `lucky_number` - Today's lucky number
- `lucky_color` - Auspicious color
- `compatibility` - Best match sign

#### Available Signs

aries, taurus, gemini, cancer, leo, virgo, libra, scorpio, sagittarius, capricorn, aquarius, pisces

AI

## Try the AI Chatbot (with API key)

The AI endpoint is what makes Vedika unique — ask any astrology question in natural language. Requires an [API key](https://vedika.io/dashboard).
```
curl -X POST https://api.vedika.io/api/v1/astrology/query \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $VEDIKA_API_KEY" \
  -d '{
    "question": "What are my career prospects?",
    "birthDetails": {
      "datetime": "1990-05-15T14:30:00",
      "latitude": 19.076,
      "longitude": 72.877,
      "timezone": "+05:30"
    }
  }'
```

#### Key Fields

- `question` — Any astrology question in natural language
- `birthDetails.datetime` — ISO 8601 format (YYYY-MM-DDTHH:MM:SS)
- `birthDetails.timezone` — UTC offset like "+05:30". IANA names like "Asia/Kolkata" also work
- `birthDetails.latitude/longitude` — Birth place coordinates

Tip:** Sandbox flat fields (`date`, `time`) also work on the live API — just add `question` and your API key header.

4

## Build Your App!

You're ready to build! Here's a complete example:
```
import requests

class VedikaClient:
    """Simple Vedika API client"""

    def __init__(self, api_key=None):
        # Use sandbox for development, production for live data
        self.base_url = "https://api.vedika.io/sandbox" if not api_key else "https://api.vedika.io"
        self.api_key = api_key

    def _headers(self):
        headers = {"Content-Type": "application/json"}
        if self.api_key:
            headers["x-api-key"] = self.api_key
        return headers

    def get_horoscope(self, sign, language="en"):
        """Get daily horoscope for a zodiac sign"""
        response = requests.get(
            f"{self.base_url}/sandbox/horoscope/daily",
            params={"sign": sign, "language": language},
            headers=self._headers()
        )
        return response.json()

    def get_birth_chart(self, datetime_str, latitude, longitude, timezone="+05:30"):
        """Generate a birth chart (Kundali)"""
        response = requests.post(
            f"{self.base_url}/v2/astrology/birth-chart",
            json={
                "datetime": datetime_str,
                "latitude": latitude,
                "longitude": longitude,
                "timezone": timezone
            },
            headers=self._headers()
        )
        return response.json()

    def ask_ai(self, question, birth_details=None):
        """Ask the AI astrologer a question"""
        payload = {"question": question}
        if birth_details:
            payload["birthDetails"] = birth_details

        response = requests.post(
            f"{self.base_url}/api/v1/astrology/query",
            json=payload,
            headers=self._headers()
        )
        return response.json()

# Usage
client = VedikaClient()  # Sandbox mode (no API key)

# Get horoscope
horoscope = client.get_horoscope("leo")
print(horoscope["horoscope"])

# Ask AI (requires API key)
client = VedikaClient(api_key="vk_live_your_key_here")
answer = client.ask_ai(
    "What does Mercury retrograde mean for my career?",
    {"datetime": "1990-01-15T14:30:00", "latitude": 19.076, "longitude": 72.877, "timezone": "+05:30"}
)
print(answer["response"])
```

## What's Next?

[

### Full Documentation

Explore all 60+ endpoints with detailed examples.

](https://vedika.io/docs) [

### API Explorer

Interactive playground to test all endpoints live.

](https://vedika.io/api-explorer) [

### Go Production

Plans from $12/mo. Real astronomical data.

](https://vedika.io/pricing)

[ ** Run in Postman ](https://www.postman.com/vedikaai/workspace/intelligence-platform/collection/50475884-27902efa-15c1-4d70-8dd3-72e874cc9be0)

---
_Full page (always current): [https://vedika.io/quickstart](https://vedika.io/quickstart) · [Vedika docs](https://vedika.io/docs)_
