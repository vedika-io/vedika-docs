---
title: "Kundli Matching API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/kundli-match
canonical: https://vedika.io/docs/sdk/kundli-match
---

# Kundli Matching API

Horoscope compatibility for marriage

POST `/v2/astrology/kundli-matching` **$0.005/call

## Basic Usage

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

const match = await vedika.kundliMatch({
  bride: {
    datetime: '1995-03-20T14:30:00+05:30',
    latitude: 28.6139,
    longitude: 77.2090
  },
  groom: {
    datetime: '1992-07-15T06:00:00+05:30',
    latitude: 19.0760,
    longitude: 72.8777
  }
});

console.log('Total Score:', match.totalScore);     // 28 out of 36
console.log('Percentage:', match.percentage);       // 77.78%
console.log('Verdict:', match.verdict);             // 'Highly Compatible'
```

## Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `bride` | BirthDetails | Yes | Bride's birth details (datetime, lat, lng) |
| `groom` | BirthDetails | Yes | Groom's birth details (datetime, lat, lng) |
| `system` | string | No | 'ashtakoot' (N. India), 'porutham' (S. India) |
| `includeRemedies` | boolean | No | Include remedies for low-scoring areas |

## Response Structure

```
{
  "totalScore": 28,
  "maxScore": 36,
  "percentage": 77.78,
  "verdict": "Highly Compatible",

  "gunas": [
    {
      "name": "Varna",
      "description": "Spiritual compatibility",
      "maxPoints": 1,
      "obtainedPoints": 1,
      "brideValue": "Brahmin",
      "groomValue": "Kshatriya",
      "isMatched": true
    },
    {
      "name": "Vashya",
      "description": "Mutual attraction",
      "maxPoints": 2,
      "obtainedPoints": 2,
      "brideValue": "Chatushpada",
      "groomValue": "Chatushpada",
      "isMatched": true
    },
    {
      "name": "Tara",
      "description": "Birth star compatibility",
      "maxPoints": 3,
      "obtainedPoints": 3,
      "brideValue": "Bharani",
      "groomValue": "Pushya",
      "isMatched": true
    },
    {
      "name": "Yoni",
      "description": "Sexual compatibility",
      "maxPoints": 4,
      "obtainedPoints": 3,
      "brideValue": "Elephant",
      "groomValue": "Goat",
      "isMatched": false
    },
    {
      "name": "Graha Maitri",
      "description": "Mental compatibility",
      "maxPoints": 5,
      "obtainedPoints": 5,
      "brideValue": "Venus",
      "groomValue": "Moon",
      "isMatched": true
    },
    {
      "name": "Gana",
      "description": "Temperament",
      "maxPoints": 6,
      "obtainedPoints": 6,
      "brideValue": "Deva",
      "groomValue": "Deva",
      "isMatched": true
    },
    {
      "name": "Bhakoot",
      "description": "Love and affection",
      "maxPoints": 7,
      "obtainedPoints": 7,
      "brideRashi": "Aries",
      "groomRashi": "Cancer",
      "isMatched": true
    },
    {
      "name": "Nadi",
      "description": "Health and genes",
      "maxPoints": 8,
      "obtainedPoints": 8,
      "brideNadi": "Aadi",
      "groomNadi": "Madhya",
      "isMatched": true
    }
  ],

  "manglikStatus": {
    "brideIsManglik": false,
    "groomIsManglik": true,
    "isCompatible": true,
    "notes": "Groom has partial Manglik dosha cancelled by Saturn aspect"
  },

  "recommendations": {
    "overallCompatibility": "excellent",
    "strengths": [
      "Strong mental and emotional connection",
      "Good financial harmony",
      "Compatible temperaments"
    ],
    "concerns": [
      "Minor differences in physical compatibility"
    ],
    "remedies": [
      "Perform Nadi Dosha Shanti puja before marriage",
      "Bride should donate rice on Saturdays"
    ]
  }
}
```

## Ashtakoot Gunas Explained

| Guna | Points | Evaluates |
| --- | --- | --- |
| **Varna** | 1 | Spiritual and ego compatibility |
| **Vashya** | 2 | Power dynamics and mutual attraction |
| **Tara** | 3 | Health and well-being after marriage |
| **Yoni** | 4 | Sexual and physical compatibility |
| **Graha Maitri** | 5 | Mental wavelength and friendship |
| **Gana** | 6 | Temperament and behavior |
| **Bhakoot** | 7 | Love, family happiness, finances |
| **Nadi** | 8 | Health of progeny, genetic compatibility |

**Scoring Guide:**
- 18+ points: Minimum recommended for marriage
- 25+ points: Good compatibility
- 32+ points: Excellent match

## South Indian (Porutham) System

```
// Use Dasa Porutham system (10 poruthams)
const match = await vedika.kundliMatch({
  bride: brideDetails,
  groom: groomDetails,
  system: 'porutham'  // South Indian method
});

// Response includes 10 poruthams
console.log(match.poruthams);
// [
//   { name: 'Dina', matched: true },
//   { name: 'Gana', matched: true },
//   { name: 'Mahendra', matched: false },
//   { name: 'Stree Deergha', matched: true },
//   { name: 'Yoni', matched: true },
//   { name: 'Rasi', matched: true },
//   { name: 'Rasiadhipati', matched: true },
//   { name: 'Vasya', matched: true },
//   { name: 'Rajju', matched: true },
//   { name: 'Vedha', matched: true }
// ]
```

## Manglik Compatibility Check

```
// Standalone Manglik check
const brideDosha = await vedika.mangalDosha({
  datetime: '1995-03-20T14:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

const groomDosha = await vedika.mangalDosha({
  datetime: '1992-07-15T06:00:00+05:30',
  latitude: 19.0760,
  longitude: 72.8777
});

// Check compatibility
if (brideDosha.isManglik && !groomDosha.isManglik) {
  console.log('Manglik mismatch - check for cancellations');
  console.log('Cancellation factors:', brideDosha.cancellations);
}
```

## Complete Matching Report

```
async function generateMatchingReport(bride, groom) {
  const vedika = new VedikaClient();

  // Get complete matching analysis
  const match = await vedika.kundliMatch({
    bride,
    groom,
    system: 'ashtakoot',
    includeRemedies: true
  });

  // Build comprehensive report
  const report = {
    summary: {
      score: `${match.totalScore}/${match.maxScore}`,
      percentage: `${match.percentage}%`,
      verdict: match.verdict
    },

    gunaBreakdown: match.gunas.map(g => ({
      guna: g.name,
      score: `${g.obtainedPoints}/${g.maxPoints}`,
      status: g.isMatched ? 'Matched' : 'Not Matched'
    })),

    concerns: match.gunas
      .filter(g => !g.isMatched)
      .map(g => `${g.name}: ${g.description}`),

    manglikAnalysis: {
      brideStatus: match.manglikStatus.brideIsManglik ? 'Manglik' : 'Non-Manglik',
      groomStatus: match.manglikStatus.groomIsManglik ? 'Manglik' : 'Non-Manglik',
      compatible: match.manglikStatus.isCompatible
    },

    recommendation: match.totalScore >= 25
      ? 'This match is recommended for marriage'
      : 'Consult an astrologer for detailed analysis',

    remedies: match.recommendations.remedies
  };

  return report;
}

// Usage
const report = await generateMatchingReport(
  { datetime: '1995-03-20T14:30:00+05:30', latitude: 28.6139, longitude: 77.2090 },
  { datetime: '1992-07-15T06:00:00+05:30', latitude: 19.0760, longitude: 72.8777 }
);

console.log(JSON.stringify(report, null, 2));
```

## Related APIs

[ **

#### Birth Chart

Individual horoscopes

](https://vedika.io/docs/sdk/birth-chart) [ **

#### Dosha Detection

Mangal, Kaal Sarp doshas

](https://vedika.io/docs/sdk/dosha) [ **

#### Matching Portal

Build matrimonial sites

](https://vedika.io/docs/sdk/matching-portal)

---
_Full page (always current): [https://vedika.io/docs/sdk/kundli-match](https://vedika.io/docs/sdk/kundli-match) · [Vedika docs](https://vedika.io/docs)_
