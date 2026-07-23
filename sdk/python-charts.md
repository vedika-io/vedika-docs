---
title: "Python Chart Generation | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/python-charts
canonical: https://vedika.io/docs/sdk/python-charts
---

# Chart Generation

Birth charts and divisional charts in Python

## Birth Chart (Rashi)

```
from vedika import VedikaClient

client = VedikaClient()

chart = client.birth_chart({
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090,
    "timezone": "Asia/Kolkata"
})

# Access chart data
print(f"Sun Sign: {chart.sun_sign}")
print(f"Moon Sign: {chart.moon_sign}")
print(f"Ascendant: {chart.ascendant}")
print(f"Nakshatra: {chart.nakshatra}")

# Planetary positions
for planet in chart.planets:
    print(f"{planet.name}: {planet.sign} at {planet.degree:.2f}deg")

# House analysis
for house in chart.houses:
    print(f"House {house.number}: {house.sign}")
```

## Divisional Charts (Vargas)

```
# Navamsha (D9) - Marriage & Spirituality
navamsha = client.divisional_chart({
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090,
    "division": "D9"
})

# Dashamsha (D10) - Career
dashamsha = client.divisional_chart({
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090,
    "division": "D10"
})

# Available divisions
# D1: Rashi (main chart)
# D2: Hora (wealth)
# D3: Drekkana (siblings)
# D4: Chaturthamsha (property)
# D7: Saptamsha (children)
# D9: Navamsha (marriage)
# D10: Dashamsha (career)
# D12: Dwadashamsha (parents)
# D16: Shodashamsha (vehicles)
# D20: Vimshamsha (spiritual)
# D24: Chaturvimshamsha (education)
# D27: Saptavimshamsha (strength)
# D30: Trimshamsha (misfortune)
# D40: Khavedamsha (auspicious)
# D45: Akshavedamsha (character)
# D60: Shashtiamsha (past karma)
```

## Chart Options

```
chart = client.birth_chart({
    "datetime": "1990-05-15T10:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090,
    "ayanamsa": "lahiri",        # 'lahiri', 'raman', 'krishnamurti'
    "house_system": "placidus",  # 'placidus', 'whole_sign', 'equal'
    "include_aspects": True,     # Include planetary aspects
    "include_yogas": True,       # Include yoga formations
    "language": "en"             # Response language
})
```

## Planetary Aspects

```
chart = client.birth_chart(birth_details, include_aspects=True)

# Access aspects
for aspect in chart.aspects:
    print(f"{aspect.planet1} {aspect.type} {aspect.planet2} (orb: {aspect.orb}deg)")

# Example output:
# Sun conjunction Mercury (orb: 3.2deg)
# Moon trine Jupiter (orb: 5.1deg)
# Mars square Saturn (orb: 2.8deg)
```

## Yoga Detection

```
chart = client.birth_chart(birth_details, include_yogas=True)

# Access yogas
for yoga in chart.yogas:
    print(f"{yoga.name}: {yoga.description}")
    print(f"  Planets: {', '.join(yoga.planets)}")
    print(f"  Strength: {yoga.strength}")

# Example output:
# Gaja Kesari Yoga: Jupiter in kendra from Moon
#   Planets: Jupiter, Moon
#   Strength: strong
```

## Panchang (Daily Calendar)

```
from datetime import date

panchang = client.panchang({
    "date": date.today().isoformat(),
    "latitude": 28.6139,
    "longitude": 77.2090,
    "timezone": "Asia/Kolkata"
})

print(f"Tithi: {panchang.tithi.name}")
print(f"Nakshatra: {panchang.nakshatra.name}")
print(f"Yoga: {panchang.yoga.name}")
print(f"Karana: {panchang.karana.name}")
print(f"Sunrise: {panchang.sun_times.sunrise}")
print(f"Rahu Kaal: {panchang.inauspicious_times.rahu_kaal}")
```

## Batch Processing

```
# Process multiple charts efficiently
birth_data_list = [
    {"datetime": "1990-05-15T10:30:00+05:30", "latitude": 28.6139, "longitude": 77.2090},
    {"datetime": "1985-03-20T14:00:00+05:30", "latitude": 19.0760, "longitude": 72.8777},
    {"datetime": "1992-08-10T06:15:00+05:30", "latitude": 13.0827, "longitude": 80.2707},
]

charts = []
for birth_data in birth_data_list:
    chart = client.birth_chart(birth_data)
    charts.append({
        "sun_sign": chart.sun_sign,
        "moon_sign": chart.moon_sign,
        "ascendant": chart.ascendant
    })

# For high volume, use async client
from vedika import AsyncVedikaClient
import asyncio

async def batch_charts(birth_data_list):
    async with AsyncVedikaClient() as client:
        tasks = [client.birth_chart(bd) for bd in birth_data_list]
        return await asyncio.gather(*tasks)
```

## Next Steps

[ **

#### Kundli Matching

Marriage compatibility

](https://vedika.io/docs/sdk/python-matching) [ **

#### Async Usage

High-performance batch

](https://vedika.io/docs/sdk/python-async)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-charts](https://vedika.io/docs/sdk/python-charts) · [Vedika docs](https://vedika.io/docs)_
