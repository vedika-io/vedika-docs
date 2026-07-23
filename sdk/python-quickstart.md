---
title: "Python SDK Quickstart | Vedika API Documentation"
source: https://vedika.io/docs/sdk/python-quickstart
canonical: https://vedika.io/docs/sdk/python-quickstart
---

# Python Quickstart

Your first API call in 5 minutes

## 1. Install the SDK

```
pip install vedika-sdk
```

## 2. Initialize the Client

```
from vedika import VedikaClient

# Option 1: Pass API key directly
client = VedikaClient(api_key="vk_live_your_api_key")

# Option 2: Use environment variable (recommended)
import os
os.environ["VEDIKA_API_KEY"] = "vk_live_your_api_key"
client = VedikaClient()
```

## 3. Make Your First Call

```
# Get birth chart
chart = client.birth_chart({
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090
})

print(f"Sun Sign: {chart.sun_sign}")      # 'Taurus'
print(f"Moon Sign: {chart.moon_sign}")    # 'Aries'
print(f"Ascendant: {chart.ascendant}")    # 'Cancer'
print(f"Nakshatra: {chart.nakshatra}")    # 'Ashwini'
```

## Complete Working Example

```
#!/usr/bin/env python3
"""quickstart.py - Vedika SDK Quickstart"""

import os
from datetime import date
from vedika import VedikaClient
from vedika.exceptions import VedikaError

def main():
    # Initialize client
    client = VedikaClient(api_key=os.environ.get("VEDIKA_API_KEY"))

    birth_details = {
        "datetime": "1990-05-15T10:30:00+05:30",
        "latitude": 28.6139,
        "longitude": 77.2090,
        "timezone": "Asia/Kolkata"
    }

    try:
        # 1. Get Birth Chart
        print("Fetching birth chart...")
        chart = client.birth_chart(birth_details)
        print(f"Birth Chart: Sun={chart.sun_sign}, Moon={chart.moon_sign}")

        # 2. Get Today's Panchang
        print("\nFetching today's panchang...")
        panchang = client.panchang({
            "date": date.today().isoformat(),
            "latitude": 28.6139,
            "longitude": 77.2090
        })
        print(f"Panchang: {panchang.tithi.name} - {panchang.nakshatra.name}")

        # 3. Get Dasha Periods
        print("\nFetching dasha periods...")
        dasha = client.vimshottari_dasha(birth_details)
        print(f"Current Mahadasha: {dasha.current_mahadasha.planet}")
        print(f"Current Antardasha: {dasha.current_antardasha.planet}")

        # 4. AI-Powered Question
        print("\nAsking AI about career...")
        response = client.query({
            "question": "What are the career prospects?",
            "birth_details": birth_details
        })
        print(f"AI Response: {response.answer[:200]}...")

    except VedikaError as e:
        print(f"API Error: {e.code} - {e.message}")

if __name__ == "__main__":
    main()
```

## Run the Example

```
# Set API key
export VEDIKA_API_KEY=vk_live_your_key

# Run script
python quickstart.py
```

Fetching birth chart...

Birth Chart: Sun=Taurus, Moon=Aries

Fetching today's panchang...

Panchang: Shukla Saptami - Pushya

Fetching dasha periods...

Current Mahadasha: Venus

Current Antardasha: Mercury

## SDK Methods Overview

| Method | Description | Pricing |
| --- | --- | --- |
| `birth_chart()` | Generate complete birth chart | $0.002 |
| `panchang()` | Daily Hindu calendar | $0.001 |
| `kundli_match()` | Marriage compatibility | $0.005 |
| `vimshottari_dasha()` | Dasha period calculation | $0.003 |
| `mangal_dosha()` | Manglik check | $0.002 |
| `query()` | AI-powered questions | $0.01-0.05 |

## Next Steps

[ **

#### Chart APIs

Birth charts in depth

](https://vedika.io/docs/sdk/python-charts) [ **

#### Async Usage

asyncio support

](https://vedika.io/docs/sdk/python-async) [ **

#### CLI Tool

Command line usage

](https://vedika.io/docs/sdk/python-cli)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-quickstart](https://vedika.io/docs/sdk/python-quickstart) · [Vedika docs](https://vedika.io/docs)_
