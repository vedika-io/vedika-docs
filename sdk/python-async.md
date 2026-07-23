---
title: "Python Async Usage | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/python-async
canonical: https://vedika.io/docs/sdk/python-async
---

# Async Usage

High-performance concurrent API calls

## Installation

```
# Install with async support
pip install vedika-sdk[async]
```

## Basic Async Client

```
import asyncio
from vedika import AsyncVedikaClient

async def main():
    # Use as context manager (recommended)
    async with AsyncVedikaClient() as client:
        chart = await client.birth_chart({
            "datetime": "1990-05-15T10:30:00+05:30",
            "latitude": 28.6139,
            "longitude": 77.2090
        })
        print(f"Sun Sign: {chart.sun_sign}")

# Run async function
asyncio.run(main())
```

## Concurrent Requests

```
import asyncio
from vedika import AsyncVedikaClient

async def fetch_multiple_charts(birth_data_list):
    """Fetch multiple charts concurrently."""
    async with AsyncVedikaClient() as client:
        # Create tasks for all requests
        tasks = [
            client.birth_chart(birth_data)
            for birth_data in birth_data_list
        ]

        # Execute all concurrently
        charts = await asyncio.gather(*tasks)

    return charts

# Example usage
birth_data = [
    {"datetime": "1990-05-15T10:30:00+05:30", "latitude": 28.6139, "longitude": 77.2090},
    {"datetime": "1985-03-20T14:00:00+05:30", "latitude": 19.0760, "longitude": 72.8777},
    {"datetime": "1992-08-10T06:15:00+05:30", "latitude": 13.0827, "longitude": 80.2707},
    {"datetime": "1988-12-25T22:30:00+05:30", "latitude": 22.5726, "longitude": 88.3639},
]

charts = asyncio.run(fetch_multiple_charts(birth_data))
for i, chart in enumerate(charts):
    print(f"Chart {i+1}: {chart.sun_sign} / {chart.moon_sign}")
```

## Rate Limiting with Semaphore

```
import asyncio
from vedika import AsyncVedikaClient

async def fetch_with_rate_limit(birth_data_list, max_concurrent=10):
    """Limit concurrent requests to avoid rate limiting."""
    semaphore = asyncio.Semaphore(max_concurrent)

    async def fetch_one(client, birth_data):
        async with semaphore:
            return await client.birth_chart(birth_data)

    async with AsyncVedikaClient() as client:
        tasks = [
            fetch_one(client, bd)
            for bd in birth_data_list
        ]
        return await asyncio.gather(*tasks)

# Process 100 charts, max 10 at a time
large_dataset = [{"datetime": f"199{i%10}-0{(i%9)+1}-{10+i%20}T10:30:00+05:30",
                  "latitude": 28.6139, "longitude": 77.2090}
                 for i in range(100)]

charts = asyncio.run(fetch_with_rate_limit(large_dataset, max_concurrent=10))
```

## Async Streaming

```
import asyncio
from vedika import AsyncVedikaClient

async def stream_response():
    async with AsyncVedikaClient() as client:
        stream = await client.query_stream({
            "question": "Analyze my career prospects",
            "birth_details": {
                "datetime": "1990-05-15T10:30:00+05:30",
                "latitude": 28.6139,
                "longitude": 77.2090
            }
        })

        async for event in stream:
            if event.type == "text":
                print(event.text, end="", flush=True)
            elif event.type == "done":
                print("\n--- Complete ---")

asyncio.run(stream_response())
```

## FastAPI Integration

```
from fastapi import FastAPI, HTTPException
from vedika import AsyncVedikaClient
from pydantic import BaseModel

app = FastAPI()

class BirthDetails(BaseModel):
    datetime: str
    latitude: float
    longitude: float

# Create client once, reuse across requests
vedika_client = AsyncVedikaClient()

@app.on_event("startup")
async def startup():
    await vedika_client.__aenter__()

@app.on_event("shutdown")
async def shutdown():
    await vedika_client.__aexit__(None, None, None)

@app.post("/chart")
async def get_chart(birth_details: BirthDetails):
    try:
        chart = await vedika_client.birth_chart(birth_details.dict())
        return {
            "sun_sign": chart.sun_sign,
            "moon_sign": chart.moon_sign,
            "ascendant": chart.ascendant
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/match")
async def match_charts(bride: BirthDetails, groom: BirthDetails):
    match = await vedika_client.kundli_match({
        "bride": bride.dict(),
        "groom": groom.dict()
    })
    return {
        "score": match.total_score,
        "verdict": match.verdict
    }
```

## Async Error Handling

```
import asyncio
from vedika import AsyncVedikaClient
from vedika.exceptions import VedikaError, RateLimitError

async def fetch_with_retry(client, birth_data, max_retries=3):
    """Fetch with automatic retry on failure."""
    for attempt in range(max_retries):
        try:
            return await client.birth_chart(birth_data)
        except RateLimitError as e:
            if attempt < max_retries - 1:
                wait_time = e.retry_after or (2 ** attempt)
                print(f"Rate limited. Waiting {wait_time}s...")
                await asyncio.sleep(wait_time)
            else:
                raise
        except VedikaError as e:
            if e.retryable and attempt < max_retries - 1:
                await asyncio.sleep(2 ** attempt)
            else:
                raise

async def main():
    async with AsyncVedikaClient() as client:
        try:
            chart = await fetch_with_retry(client, birth_data)
            print(f"Success: {chart.sun_sign}")
        except VedikaError as e:
            print(f"Failed after retries: {e.message}")
```

## Performance Comparison

| Method | 10 Charts | 100 Charts |
| --- | --- | --- |
| Sync (sequential) | ~5 seconds | ~50 seconds |
| Async (concurrent) | ~0.6 seconds | ~6 seconds |

* Based on ~500ms average API response time

## Related Topics

[ **

#### Django Integration

Django Channels async

](https://vedika.io/docs/sdk/django-integration) [ **

#### CLI Tool

Command line usage

](https://vedika.io/docs/sdk/python-cli)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-async](https://vedika.io/docs/sdk/python-async) · [Vedika docs](https://vedika.io/docs)_
