---
title: "Birth Chart API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/birth-chart
canonical: https://vedika.io/docs/sdk/birth-chart
---

# Birth Chart API

Generate complete Vedic birth charts (Kundli)

POST `/v2/astrology/birth-chart` **$0.002/call

## Basic Usage

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

const chart = await vedika.birthChart({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  timezone: 'Asia/Kolkata'
});

console.log(chart.sunSign);     // 'Taurus'
console.log(chart.moonSign);    // 'Aries'
console.log(chart.ascendant);   // 'Cancer'
console.log(chart.nakshatra);   // 'Ashwini'
```

## Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `datetime` | string | Yes | ISO 8601 datetime with timezone offset |
| `latitude` | number | Yes | Birth location latitude (-90 to 90) |
| `longitude` | number | Yes | Birth location longitude (-180 to 180) |
| `timezone` | string | No | IANA timezone (e.g., 'Asia/Kolkata') |
| `ayanamsa` | string | No | 'lahiri' (default), 'krishnamurti', 'raman' |
| `houseSystem` | string | No | 'placidus' (default), 'whole-sign', 'equal' |

## Response Structure

```
{
  "sunSign": "Taurus",
  "moonSign": "Aries",
  "ascendant": "Cancer",
  "nakshatra": "Ashwini",
  "nakshatraPada": 2,
  "nakshatraLord": "Ketu",

  "planets": [
    {
      "name": "Sun",
      "sign": "Taurus",
      "signLord": "Venus",
      "house": 11,
      "degree": 24.38,
      "nakshatra": "Rohini",
      "nakshatraPada": 3,
      "isRetrograde": false,
      "isCombust": false,
      "isExalted": false,
      "isDebilitated": false
    },
    // ... other planets
  ],

  "houses": [
    { "number": 1, "sign": "Cancer", "signLord": "Moon", "degree": 15.2 },
    { "number": 2, "sign": "Leo", "signLord": "Sun", "degree": 12.4 },
    // ... houses 3-12
  ],

  "aspects": [
    {
      "planet1": "Sun",
      "planet2": "Moon",
      "aspectType": "sextile",
      "orb": 2.3
    },
    // ... other aspects
  ],

  "yogas": [
    {
      "name": "Gaja Kesari Yoga",
      "planets": ["Jupiter", "Moon"],
      "description": "Jupiter in kendra from Moon",
      "strength": "strong"
    }
  ],

  "metadata": {
    "ayanamsa": "lahiri",
    "ayanamsaValue": 24.12,
    "julianDay": 2448029.94,
    "siderealTime": "03:45:12"
  }
}
```

## Advanced Options

```
// With all options
const chart = await vedika.birthChart({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  timezone: 'Asia/Kolkata',
  ayanamsa: 'lahiri',           // Ayanamsa system
  houseSystem: 'placidus',      // House calculation method
  includeAspects: true,         // Include planetary aspects
  includeYogas: true,           // Include yoga formations
  includeNakshatras: true,      // Include nakshatra details
  language: 'en'                // Response language
});
```

## Divisional Charts (Vargas)

Get divisional charts like D9 (Navamsha), D10 (Dashamsha):
```
// Get Navamsha chart (D9)
const navamsha = await vedika.divisionalChart({
  ...birthDetails,
  division: 'D9'  // Navamsha
});

// Get Dashamsha chart (D10) for career
const dashamsha = await vedika.divisionalChart({
  ...birthDetails,
  division: 'D10'  // Dashamsha
});

// Available divisions: D1-D60
// Common: D9 (marriage), D10 (career), D7 (children), D4 (property)
```

## TypeScript Types

```
import {
  VedikaClient,
  BirthChart,
  BirthChartRequest,
  Planet,
  House,
  Aspect
} from '@vedika-io/sdk';

const request: BirthChartRequest = {
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
};

const chart: BirthChart = await vedika.birthChart(request);

// Access typed properties
const sun: Planet = chart.planets.find(p => p.name === 'Sun')!;
console.log(sun.sign);  // Type: ZodiacSign

const firstHouse: House = chart.houses[0];
console.log(firstHouse.signLord);  // Type: PlanetName
```

## Real-World Example

```
// Complete birth chart analysis
async function analyzeBirthChart(birthDetails) {
  const vedika = new VedikaClient();

  // Get main birth chart
  const chart = await vedika.birthChart(birthDetails);

  // Build analysis object
  const analysis = {
    // Basic signs
    personality: {
      sunSign: chart.sunSign,
      moonSign: chart.moonSign,
      risingSign: chart.ascendant
    },

    // Planet positions by house
    planetsByHouse: chart.planets.reduce((acc, planet) => {
      acc[planet.house] = acc[planet.house] || [];
      acc[planet.house].push(planet.name);
      return acc;
    }, {}),

    // Find strong planets
    strongPlanets: chart.planets.filter(p =>
      p.isExalted || p.house === 1 || p.house === 10
    ).map(p => p.name),

    // Find weak planets
    challengingPlanets: chart.planets.filter(p =>
      p.isDebilitated || p.isCombust
    ).map(p => p.name),

    // Active yogas
    beneficYogas: chart.yogas.filter(y =>
      y.strength === 'strong'
    ).map(y => y.name)
  };

  return analysis;
}

// Usage
const result = await analyzeBirthChart({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log('Analysis:', result);
```

## Related APIs

[ **

#### Kundli Matching

Compare two charts

](https://vedika.io/docs/sdk/kundli-match) [ **

#### Dasha Periods

Planetary periods

](https://vedika.io/docs/sdk/dasha) [ **

#### Dosha Detection

Check for doshas

](https://vedika.io/docs/sdk/dosha)

---
_Full page (always current): [https://vedika.io/docs/sdk/birth-chart](https://vedika.io/docs/sdk/birth-chart) · [Vedika docs](https://vedika.io/docs)_
