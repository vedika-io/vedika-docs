---
title: "Python Authentication | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/python-auth
canonical: https://vedika.io/docs/sdk/python-auth
---

# Python Authentication

API key management in Python

## Direct API Key

```
from vedika import VedikaClient

# Pass API key directly
client = VedikaClient(api_key="vk_live_abc123xyz789")
```

Never hardcode API keys in source code. Use environment variables instead.

## Environment Variables (Recommended)

```
# .env file
VEDIKA_API_KEY=vk_live_abc123xyz789
```

```
# Using python-dotenv
from dotenv import load_dotenv
from vedika import VedikaClient

load_dotenv()  # Load .env file

# Client auto-reads VEDIKA_API_KEY
client = VedikaClient()
```

## Django Integration

```
# settings.py
import os

VEDIKA_API_KEY = os.environ.get("VEDIKA_API_KEY")

# views.py
from django.conf import settings
from vedika import VedikaClient

def get_vedika_client():
    return VedikaClient(api_key=settings.VEDIKA_API_KEY)
```

## Flask Integration

```
from flask import Flask, g
from vedika import VedikaClient

app = Flask(__name__)
app.config.from_prefixed_env()  # Reads FLASK_VEDIKA_API_KEY

def get_client():
    if 'vedika_client' not in g:
        g.vedika_client = VedikaClient(
            api_key=app.config.get('VEDIKA_API_KEY')
        )
    return g.vedika_client

@app.route('/chart')
def birth_chart():
    client = get_client()
    # Use client...
```

## Test vs Live Keys

```
import os
from vedika import VedikaClient

# Use test key in development, live in production
api_key = os.environ.get("VEDIKA_API_KEY")

if api_key.startswith("vk_test_"):
    print("Using TEST mode - mock data returned")
elif api_key.startswith("vk_live_"):
    print("Using LIVE mode - real calculations")

client = VedikaClient(api_key=api_key)
```

## Verify Connection

```
from vedika import VedikaClient
from vedika.exceptions import VedikaError

client = VedikaClient()

try:
    health = client.health()
    print(f"API Status: {health.status}")

    usage = client.usage()
    print(f"Credits: {usage.credits_remaining}")
except VedikaError as e:
    if e.code == "INVALID_API_KEY":
        print("Invalid API key!")
    else:
        print(f"Error: {e.message}")
```

## Next Steps

[ **

#### Chart APIs

Generate birth charts

](https://vedika.io/docs/sdk/python-charts) [ **

#### Flask Guide

Full Flask integration

](https://vedika.io/docs/sdk/flask-integration)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-auth](https://vedika.io/docs/sdk/python-auth) · [Vedika docs](https://vedika.io/docs)_
