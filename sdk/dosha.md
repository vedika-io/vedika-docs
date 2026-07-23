---
title: "Dosha Detection API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/dosha
canonical: https://vedika.io/docs/sdk/dosha
---

# Dosha Detection API

Analyze astrological doshas with remedies

## Available Dosha Checks

Mangal Dosha**

Mars in 1st, 4th, 7th, 8th, or 12th house. Affects marriage.

**$0.002

Kaal Sarp Dosha**

All planets between Rahu and Ketu. Causes life struggles.

**$0.002

Pitru Dosha**

Ancestral karma afflictions. Sun/Moon with Rahu/Ketu.

**$0.002

Nadi Dosha**

Same Nadi in matching. Affects progeny health.

**$0.003

## Mangal Dosha (Manglik Check)

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

const mangalDosha = await vedika.mangalDosha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log('Is Manglik:', mangalDosha.isManglik);           // true
console.log('Severity:', mangalDosha.severity);             // 'partial'
console.log('Mars House:', mangalDosha.marsHouse);          // 7
console.log('Cancellations:', mangalDosha.cancellations);   // ['Saturn aspect']
```

### Response Structure

```
{
  "isManglik": true,
  "severity": "partial",  // 'none', 'partial', 'full'
  "marsPosition": {
    "house": 7,
    "sign": "Scorpio",
    "degree": 15.23,
    "isRetrograde": false
  },
  "affectedHouses": [7],  // Houses where Mars creates dosha
  "doshaType": "Seventh House Manglik",
  "effects": [
    "Delays in marriage",
    "Conflicts with spouse",
    "Possibility of divorce if not matched properly"
  ],
  "cancellations": [
    {
      "factor": "Saturn aspecting Mars",
      "description": "Saturn's 10th aspect on Mars reduces Manglik effects",
      "reductionPercentage": 50
    },
    {
      "factor": "Mars in own sign",
      "description": "Mars in Scorpio reduces negative effects",
      "reductionPercentage": 25
    }
  ],
  "effectiveSeverity": "low",  // After cancellations
  "remedies": [
    {
      "type": "puja",
      "name": "Mangal Shanti Puja",
      "description": "Perform on Tuesday with red flowers"
    },
    {
      "type": "mantra",
      "name": "Mangal Beej Mantra",
      "mantra": "Om Kraam Kreem Kraum Sah Bhaumaya Namah",
      "count": 10000
    },
    {
      "type": "gemstone",
      "name": "Red Coral",
      "finger": "Ring finger",
      "metal": "Gold or Copper"
    },
    {
      "type": "charity",
      "name": "Donate red lentils on Tuesday"
    }
  ],
  "marriageCompatibility": {
    "shouldMarryManglik": true,
    "kumbhVivahRequired": false,
    "minimumMatchScore": 18
  }
}
```

## Kaal Sarp Dosha

```
const kaalSarp = await vedika.kaalSarpDosha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log('Has Kaal Sarp:', kaalSarp.hasDosha);        // true
console.log('Type:', kaalSarp.type);                     // 'Anant Kaal Sarp'
console.log('Direction:', kaalSarp.direction);          // 'ascending'
```

### 12 Types of Kaal Sarp

| Type | Rahu House | Ketu House | Primary Effect |
| --- | --- | --- | --- |
| Anant | 1 | 7 | Health, relationships |
| Kulik | 2 | 8 | Finances, longevity |
| Vasuki | 3 | 9 | Siblings, fortune |
| Shankhpal | 4 | 10 | Property, career |
| Padma | 5 | 11 | Children, gains |
| Mahapadma | 6 | 12 | Enemies, losses |
| Takshak | 7 | 1 | Marriage, self |
| Karkotak | 8 | 2 | Sudden events, family |
| Shankachood | 9 | 3 | Fortune, courage |
| Ghatak | 10 | 4 | Career, home |
| Vishdhar | 11 | 5 | Gains, children |
| Sheshnag | 12 | 6 | Expenses, health |

## Pitru Dosha

```
const pitruDosha = await vedika.pitruDosha({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log('Has Pitru Dosha:', pitruDosha.hasDosha);
console.log('Causes:', pitruDosha.causes);
// ['Sun conjunct Rahu in 9th house', 'Moon with Ketu']

console.log('Remedies:', pitruDosha.remedies);
// ['Perform Pitru Tarpan on Amavasya', 'Donate food to Brahmins']
```

## Complete Dosha Analysis

```
// Check all doshas at once
const allDoshas = await vedika.doshaAnalysis({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090,
  includeRemedies: true
});

console.log('Dosha Summary:');
console.log('Mangal Dosha:', allDoshas.mangal.hasDosha ? 'Yes' : 'No');
console.log('Kaal Sarp Dosha:', allDoshas.kaalSarp.hasDosha ? 'Yes' : 'No');
console.log('Pitru Dosha:', allDoshas.pitru.hasDosha ? 'Yes' : 'No');

// Get combined remedies
const allRemedies = [
  ...allDoshas.mangal.remedies,
  ...allDoshas.kaalSarp.remedies,
  ...allDoshas.pitru.remedies
];

console.log('Total remedies:', allRemedies.length);
```

## Dosha Compatibility for Marriage

```
// Check if two people's doshas are compatible
const doshaCompatibility = await vedika.doshaCompatibility({
  person1: {
    datetime: '1990-05-15T10:30:00+05:30',
    latitude: 28.6139,
    longitude: 77.2090
  },
  person2: {
    datetime: '1992-08-20T14:00:00+05:30',
    latitude: 19.0760,
    longitude: 72.8777
  }
});

console.log('Compatibility:', doshaCompatibility.isCompatible);
console.log('Manglik Status:', {
  person1: doshaCompatibility.person1.isManglik,
  person2: doshaCompatibility.person2.isManglik
});
console.log('Nadi Dosha:', doshaCompatibility.nadiDosha.hasDosha);
console.log('Recommendations:', doshaCompatibility.recommendations);
```

## Generate Dosha Report

```
async function generateDoshaReport(birthDetails) {
  const vedika = new VedikaClient();

  const analysis = await vedika.doshaAnalysis({
    ...birthDetails,
    includeRemedies: true
  });

  // Build comprehensive report
  const report = {
    summary: {
      totalDoshasFound: 0,
      severityLevel: 'low',
      primaryConcern: null
    },
    doshas: [],
    prioritizedRemedies: []
  };

  // Check each dosha
  const doshaTypes = ['mangal', 'kaalSarp', 'pitru'];

  for (const type of doshaTypes) {
    if (analysis[type].hasDosha) {
      report.summary.totalDoshasFound++;
      report.doshas.push({
        name: formatDoshaName(type),
        severity: analysis[type].severity || 'moderate',
        effects: analysis[type].effects,
        remedies: analysis[type].remedies
      });
    }
  }

  // Set primary concern
  if (report.doshas.length > 0) {
    const severityOrder = { full: 3, moderate: 2, partial: 1 };
    report.doshas.sort((a, b) =>
      (severityOrder[b.severity] || 0) - (severityOrder[a.severity] || 0)
    );
    report.summary.primaryConcern = report.doshas[0].name;
    report.summary.severityLevel = report.doshas[0].severity;
  }

  // Prioritize remedies
  report.prioritizedRemedies = report.doshas
    .flatMap(d => d.remedies)
    .slice(0, 5);

  return report;
}

function formatDoshaName(type) {
  const names = {
    mangal: 'Mangal Dosha',
    kaalSarp: 'Kaal Sarp Dosha',
    pitru: 'Pitru Dosha'
  };
  return names[type] || type;
}

// Usage
const report = await generateDoshaReport({
  datetime: '1990-05-15T10:30:00+05:30',
  latitude: 28.6139,
  longitude: 77.2090
});

console.log(JSON.stringify(report, null, 2));
```

## Related APIs

[ **

#### Kundli Matching

Marriage compatibility

](https://vedika.io/docs/sdk/kundli-match) [ **

#### Birth Chart

Full horoscope

](https://vedika.io/docs/sdk/birth-chart) [ **

#### Dasha Periods

Planetary periods

](https://vedika.io/docs/sdk/dasha)

---
_Full page (always current): [https://vedika.io/docs/sdk/dosha](https://vedika.io/docs/sdk/dosha) · [Vedika docs](https://vedika.io/docs)_
