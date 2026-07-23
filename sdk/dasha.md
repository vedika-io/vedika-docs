---
title: "Dasha Periods API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/dasha
canonical: https://vedika.io/docs/sdk/dasha
---

# Dasha Periods API

Vimshottari Dasha - Planetary period calculations

POST `/v2/astrology/vimshottari-dasha` **$0.003/call

## Basic Usage

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

const dasha = await vedika.vimshottariDasha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

// Current periods
console.log('Current Mahadasha:', dasha.currentMahadasha.planet);    // 'Venus'
console.log('Current Antardasha:', dasha.currentAntardasha.planet); // 'Mercury'

// Period dates
console.log('Mahadasha ends:', dasha.currentMahadasha.endDate);     // '2032-05-15'
```

## Response Structure

```
{
  "birthNakshatra": "Ashwini",
  "birthNakshatraLord": "Ketu",
  "dashaBalance": {
    "planet": "Ketu",
    "years": 2,
    "months": 3,
    "days": 15
  },

  "currentMahadasha": {
    "planet": "Venus",
    "startDate": "2012-08-30",
    "endDate": "2032-08-30",
    "duration": "20 years",
    "interpretation": "Period of luxury, relationships, and artistic pursuits"
  },

  "currentAntardasha": {
    "planet": "Mercury",
    "startDate": "2025-04-30",
    "endDate": "2028-02-28",
    "parentMahadasha": "Venus",
    "interpretation": "Focus on communication, business, and intellectual growth"
  },

  "currentPratyantardasha": {
    "planet": "Jupiter",
    "startDate": "2026-01-01",
    "endDate": "2026-05-15",
    "interpretation": "Expansion and wisdom in Venus-Mercury period"
  },

  "mahadashas": [
    {
      "planet": "Ketu",
      "startDate": "1990-05-15",
      "endDate": "1997-05-15",
      "duration": "7 years",
      "isActive": false,
      "isPast": true
    },
    {
      "planet": "Venus",
      "startDate": "1997-05-15",
      "endDate": "2017-05-15",
      "duration": "20 years",
      "isActive": false,
      "isPast": true
    },
    {
      "planet": "Sun",
      "startDate": "2017-05-15",
      "endDate": "2023-05-15",
      "duration": "6 years",
      "isActive": false,
      "isPast": true
    },
    {
      "planet": "Moon",
      "startDate": "2023-05-15",
      "endDate": "2033-05-15",
      "duration": "10 years",
      "isActive": true,
      "isPast": false
    },
    // ... remaining periods
  ],

  "antardashas": [
    {
      "planet": "Moon",
      "startDate": "2023-05-15",
      "endDate": "2024-03-15",
      "parentMahadasha": "Moon",
      "isActive": false
    },
    {
      "planet": "Mars",
      "startDate": "2024-03-15",
      "endDate": "2024-10-15",
      "parentMahadasha": "Moon",
      "isActive": false
    },
    {
      "planet": "Rahu",
      "startDate": "2024-10-15",
      "endDate": "2026-04-15",
      "parentMahadasha": "Moon",
      "isActive": true
    },
    // ... remaining antardashas for current mahadasha
  ]
}
```

## Vimshottari Dasha Periods

| Planet | Duration | Nakshatras |
| --- | --- | --- |
| Ketu | 7 years | Ashwini, Magha, Moola |
| Venus | 20 years | Bharani, Purva Phalguni, Purva Ashadha |
| Sun | 6 years | Krittika, Uttara Phalguni, Uttara Ashadha |
| Moon | 10 years | Rohini, Hasta, Shravana |
| Mars | 7 years | Mrigashira, Chitra, Dhanishta |
| Rahu | 18 years | Ardra, Swati, Shatabhisha |
| Jupiter | 16 years | Punarvasu, Vishakha, Purva Bhadrapada |
| Saturn | 19 years | Pushya, Anuradha, Uttara Bhadrapada |
| Mercury | 17 years | Ashlesha, Jyeshtha, Revati |

## Get Specific Dasha Levels

```
// Get only Mahadashas
const mahadashas = await vedika.mahadasha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

// Get Antardashas for a specific Mahadasha
const antardashas = await vedika.antardasha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  mahadasha: 'Moon'  // Get all antardashas within Moon mahadasha
});

// Get Pratyantardashas (sub-sub periods)
const pratyantardashas = await vedika.pratyantardasha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  mahadasha: 'Moon',
  antardasha: 'Rahu'
});
```

## Find Dasha for Specific Date

```
// What dasha will be running in 2030?
const futureDasha = await vedika.dashaForDate({
  birthDetails: {
    datetime: '1990-05-15T10:30:00+05:30',
    latitude: 28.6139,
    longitude: 77.2090
  },
  targetDate: '2030-06-15'
});

console.log('In 2030:');
console.log('Mahadasha:', futureDasha.mahadasha);     // 'Moon'
console.log('Antardasha:', futureDasha.antardasha);   // 'Saturn'
console.log('Pratyantardasha:', futureDasha.pratyantardasha); // 'Venus'
```

## Dasha Predictions

```
// Get predictions for current dasha
const predictions = await vedika.dashaPredictions({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  includeRemedies: true
});

console.log('Current Period Analysis:');
console.log(predictions.currentPeriod);
// {
//   mahadasha: 'Moon',
//   antardasha: 'Rahu',
//   themes: ['emotional intensity', 'foreign connections', 'unexpected changes'],
//   career: 'Good for jobs involving public dealing, media, foreign trade',
//   relationships: 'May face misunderstandings, need patience',
//   health: 'Watch for mental stress, sleep issues',
//   finance: 'Mixed results, avoid speculation',
//   remedies: ['Chant Moon mantras on Mondays', 'Donate white items']
// }
```

## Build Timeline Visualization

```
async function buildDashaTimeline(birthDetails) {
  const vedika = new VedikaClient();

  const dasha = await vedika.vimshottariDasha(birthDetails);
  const today = new Date();

  // Build timeline data
  const timeline = dasha.mahadashas.map(md => {
    const startDate = new Date(md.startDate);
    const endDate = new Date(md.endDate);
    const isActive = today >= startDate && today <= endDate;
    const isPast = today > endDate;

    // Calculate progress percentage
    const totalDays = (endDate - startDate) / (1000 * 60 * 60 * 24);
    const elapsedDays = Math.max(0, (today - startDate) / (1000 * 60 * 60 * 24));
    const progress = isPast ? 100 : isActive ? (elapsedDays / totalDays) * 100 : 0;

    return {
      planet: md.planet,
      startDate: md.startDate,
      endDate: md.endDate,
      duration: md.duration,
      isActive,
      isPast,
      isFuture: !isActive && !isPast,
      progress: Math.round(progress),
      color: getPlanetColor(md.planet)
    };
  });

  return timeline;
}

function getPlanetColor(planet) {
  const colors = {
    Sun: '#f59e0b',
    Moon: '#e5e7eb',
    Mars: '#ef4444',
    Mercury: '#22c55e',
    Jupiter: '#eab308',
    Venus: '#ec4899',
    Saturn: '#3b82f6',
    Rahu: '#6366f1',
    Ketu: '#8b5cf6'
  };
  return colors[planet] || '#6b7280';
}

// Usage
const timeline = await buildDashaTimeline({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log(timeline);
```

## Related APIs

[ **

#### Birth Chart

Full horoscope data

](https://vedika.io/docs/sdk/birth-chart) [ **

#### Dosha Detection

Check for doshas

](https://vedika.io/docs/sdk/dosha) [ **

#### Transits API

Current planetary transits

](https://vedika.io/docs/api/v2/astrology/saturn-transit)

---
_Full page (always current): [https://vedika.io/docs/sdk/dasha](https://vedika.io/docs/sdk/dasha) · [Vedika docs](https://vedika.io/docs)_
