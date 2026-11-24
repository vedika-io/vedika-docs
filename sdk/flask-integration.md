---
title: "Flask Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/flask-integration
canonical: https://vedika.io/docs/sdk/flask-integration
---

# Flask Integration

Build astrology APIs with Flask

## Setup

```
# Create project
mkdir flask-astrology && cd flask-astrology
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install flask vedika-sdk python-dotenv
```

## Basic Flask App

```
# app.py
from flask import Flask, request, jsonify
from dotenv import load_dotenv
from vedika import VedikaClient
from vedika.exceptions import VedikaError

load_dotenv()

app = Flask(__name__)
vedika = VedikaClient()

@app.route('/api/chart', methods=['POST'])
def birth_chart():
    data = request.get_json()

    try:
        chart = vedika.birth_chart({
            'datetime': data['datetime'],
            'latitude': data['latitude'],
            'longitude': data['longitude']
        })

        return jsonify({
            'sun_sign': chart.sun_sign,
            'moon_sign': chart.moon_sign,
            'ascendant': chart.ascendant,
            'planets': [
                {
                    'name': p.name,
                    'sign': p.sign,
                    'degree': p.degree
                }
                for p in chart.planets
            ]
        })
    except VedikaError as e:
        return jsonify({'error': e.message}), 500

@app.route('/api/panchang', methods=['GET'])
def panchang():
    date = request.args.get('date')
    lat = float(request.args.get('lat', 28.6139))
    lng = float(request.args.get('lng', 77.2090))

    try:
        panchang = vedika.panchang({
            'date': date,
            'latitude': lat,
            'longitude': lng
        })

        return jsonify({
            'tithi': panchang.tithi.name,
            'nakshatra': panchang.nakshatra.name,
            'yoga': panchang.yoga.name,
            'karana': panchang.karana.name
        })
    except VedikaError as e:
        return jsonify({'error': e.message}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

## Blueprints Structure

```
# routes/astrology.py
from flask import Blueprint, request, jsonify
from services.vedika_service import VedikaService

astrology_bp = Blueprint('astrology', __name__, url_prefix='/api')
vedika_service = VedikaService()

@astrology_bp.route('/chart', methods=['POST'])
def chart():
    data = request.get_json()
    result = vedika_service.get_chart(data)
    return jsonify(result)

@astrology_bp.route('/match', methods=['POST'])
def match():
    data = request.get_json()
    result = vedika_service.match_charts(data['bride'], data['groom'])
    return jsonify(result)

@astrology_bp.route('/dasha', methods=['POST'])
def dasha():
    data = request.get_json()
    result = vedika_service.get_dasha(data)
    return jsonify(result)
```

```
# services/vedika_service.py
from vedika import VedikaClient
from vedika.exceptions import VedikaError

class VedikaService:
    def __init__(self):
        self.client = VedikaClient()

    def get_chart(self, birth_details):
        chart = self.client.birth_chart(birth_details)
        return {
            'sun_sign': chart.sun_sign,
            'moon_sign': chart.moon_sign,
            'ascendant': chart.ascendant,
            'nakshatra': chart.nakshatra
        }

    def match_charts(self, bride, groom):
        match = self.client.kundli_match({
            'bride': bride,
            'groom': groom
        })
        return {
            'score': match.total_score,
            'max_score': match.max_score,
            'verdict': match.verdict
        }

    def get_dasha(self, birth_details):
        dasha = self.client.vimshottari_dasha(birth_details)
        return {
            'current_mahadasha': dasha.current_mahadasha.planet,
            'current_antardasha': dasha.current_antardasha.planet
        }
```

## Error Handling

```
# app.py
from flask import Flask, jsonify
from vedika.exceptions import VedikaError, RateLimitError

app = Flask(__name__)

@app.errorhandler(VedikaError)
def handle_vedika_error(error):
    return jsonify({
        'error': error.message,
        'code': error.code
    }), error.status

@app.errorhandler(RateLimitError)
def handle_rate_limit(error):
    return jsonify({
        'error': 'Rate limit exceeded',
        'retry_after': error.retry_after
    }), 429

@app.errorhandler(400)
def handle_bad_request(error):
    return jsonify({'error': 'Bad request'}), 400

@app.errorhandler(500)
def handle_server_error(error):
    return jsonify({'error': 'Internal server error'}), 500
```

## Caching with Flask-Caching

```
# app.py
from flask import Flask
from flask_caching import Cache

app = Flask(__name__)
cache = Cache(app, config={
    'CACHE_TYPE': 'redis',
    'CACHE_REDIS_URL': 'redis://localhost:6379/0'
})

@app.route('/api/panchang')
@cache.cached(timeout=3600, query_string=True)  # Cache 1 hour
def panchang():
    # This will be cached
    date = request.args.get('date')
    return vedika.panchang({'date': date, ...})
```

## Async with Quart

```
# quart_app.py
from quart import Quart, request, jsonify
from vedika import AsyncVedikaClient

app = Quart(__name__)
vedika = None

@app.before_serving
async def startup():
    global vedika
    vedika = AsyncVedikaClient()
    await vedika.__aenter__()

@app.after_serving
async def shutdown():
    await vedika.__aexit__(None, None, None)

@app.route('/api/chart', methods=['POST'])
async def chart():
    data = await request.get_json()
    chart = await vedika.birth_chart(data)
    return jsonify({
        'sun_sign': chart.sun_sign,
        'moon_sign': chart.moon_sign
    })

if __name__ == '__main__':
    app.run()
```

## Production Setup

```
# Install Gunicorn
pip install gunicorn

# Run with Gunicorn
gunicorn -w 4 -b 0.0.0.0:8000 app:app

# With environment variables
VEDIKA_API_KEY=vk_live_xxx gunicorn -w 4 app:app
```

```
# config.py
import os

class Config:
    VEDIKA_API_KEY = os.environ.get('VEDIKA_API_KEY')
    CACHE_TYPE = 'redis'
    CACHE_REDIS_URL = os.environ.get('REDIS_URL')

class ProductionConfig(Config):
    DEBUG = False

class DevelopmentConfig(Config):
    DEBUG = True
```

## Next Steps

[ **

#### Django Integration

Full-stack Django

](https://vedika.io/docs/sdk/django-integration)

---
_Full page (always current): [https://vedika.io/docs/sdk/flask-integration](https://vedika.io/docs/sdk/flask-integration) · [Vedika docs](https://vedika.io/docs)_
