---
title: "Python SDK Installation | Vedika API Documentation"
source: https://vedika.io/docs/sdk/python-install
canonical: https://vedika.io/docs/sdk/python-install
---

# Python SDK Installation

vedika-sdk

**v1.8.0 Python 3.8+

## Quick Install

```
# Install via pip
pip install vedika-sdk

# Or with pip3
pip3 install vedika-sdk

# Install specific version
pip install vedika-sdk==1.8.0
```

## Recommended: Virtual Environment

```
# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install package
pip install vedika-sdk
```

## Requirements

| Python | >= 3.8 |
| --- | --- |
| Dependencies | httpx, pydantic, python-dateutil |
| Optional | aiohttp (async), rich (CLI) |

## Verify Installation

```
from vedika import VedikaClient

# Check version
print(VedikaClient.VERSION)  # '1.8.0'

# Initialize client
client = VedikaClient(api_key="your-api-key")

# Test connection
health = client.health()
print(f"API Status: {health.status}")  # 'ok'
```

## Optional Dependencies

```
# For async support
pip install vedika-sdk[async]

# For CLI tool
pip install vedika-sdk[cli]

# For all optional features
pip install vedika-sdk[all]
```

## Environment Setup

```
# .env file
VEDIKA_API_KEY=vk_live_abc123xyz789
```

```
# With python-dotenv
from dotenv import load_dotenv
from vedika import VedikaClient

load_dotenv()

# Client automatically reads VEDIKA_API_KEY from environment
client = VedikaClient()
```

## Type Hints Support

Full type hints with Pydantic models for IDE autocompletion:
```
from vedika import VedikaClient
from vedika.types import BirthChart, BirthDetails, Planet

client = VedikaClient()

birth_details: BirthDetails = {
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090
}

chart: BirthChart = client.birth_chart(birth_details)

# Full autocompletion for response properties
print(chart.sun_sign)  # IDE knows this is a string
print(chart.planets)   # IDE knows this is List[Planet]
```

## Package Contents

#### Main Exports

- `VedikaClient` - Sync client
- `AsyncVedikaClient` - Async client
- `VedikaError` - Exception class

#### Type Models

- `vedika.types.BirthChart`
- `vedika.types.Panchang`
- `vedika.types.KundliMatch`
- `vedika.types.DashaPeriod`

## Next Steps

[ **

#### Quickstart Guide

Your first API call

](https://vedika.io/docs/sdk/python-quickstart) [ **

#### Async Usage

asyncio integration

](https://vedika.io/docs/sdk/python-async)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-install](https://vedika.io/docs/sdk/python-install) · [Vedika docs](https://vedika.io/docs)_
