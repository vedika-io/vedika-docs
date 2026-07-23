---
title: "Panchang API | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/panchang
canonical: https://vedika.io/docs/sdk/panchang
---

# Panchang API

Daily Hindu calendar with Tithi, Nakshatra, Yoga, Karana

POST `/v2/astrology/panchang` **$0.001/call

## Basic Usage

```
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

// Get today's panchang for Delhi
const panchang = await vedika.panchang({
  date: '2026-01-07',
  latitude: 28.6139,
  longitude: 77.2090,
  timezone: 'Asia/Kolkata'
});

console.log('Tithi:', panchang.tithi.name);         // 'Shukla Saptami'
console.log('Nakshatra:', panchang.nakshatra.name); // 'Pushya'
console.log('Yoga:', panchang.yoga.name);           // 'Siddha'
console.log('Karana:', panchang.karana.name);       // 'Bava'
```

## Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `date` | string | Yes | Date in YYYY-MM-DD format |
| `latitude` | number | Yes | Location latitude for sunrise/sunset |
| `longitude` | number | Yes | Location longitude |
| `timezone` | string | No | IANA timezone (default: UTC) |

## Response Structure

```
{
  "date": "2026-01-07",
  "dayOfWeek": "Wednesday",
  "hinduMonth": "Pausha",
  "hinduYear": "Shobhakrit",
  "paksha": "Shukla",

  "tithi": {
    "name": "Saptami",
    "number": 7,
    "paksha": "Shukla",
    "lord": "Sun",
    "startTime": "2026-01-07T03:45:00+05:30",
    "endTime": "2026-01-08T02:12:00+05:30",
    "isAuspicious": true
  },

  "nakshatra": {
    "name": "Pushya",
    "number": 8,
    "lord": "Saturn",
    "deity": "Brihaspati",
    "startTime": "2026-01-07T01:23:00+05:30",
    "endTime": "2026-01-08T00:45:00+05:30",
    "pada": 2
  },

  "yoga": {
    "name": "Siddha",
    "number": 21,
    "meaning": "Accomplishment",
    "startTime": "2026-01-07T05:30:00+05:30",
    "endTime": "2026-01-08T04:15:00+05:30",
    "isAuspicious": true
  },

  "karana": {
    "name": "Bava",
    "number": 1,
    "startTime": "2026-01-07T03:45:00+05:30",
    "endTime": "2026-01-07T14:30:00+05:30"
  },

  "sunTimes": {
    "sunrise": "2026-01-07T07:14:00+05:30",
    "sunset": "2026-01-07T17:38:00+05:30",
    "moonrise": "2026-01-07T11:45:00+05:30",
    "moonset": "2026-01-08T00:23:00+05:30"
  },

  "auspiciousTimes": {
    "abhijitMuhurta": {
      "start": "2026-01-07T11:54:00+05:30",
      "end": "2026-01-07T12:38:00+05:30"
    },
    "brahmaMuhurta": {
      "start": "2026-01-07T05:26:00+05:30",
      "end": "2026-01-07T06:14:00+05:30"
    }
  },

  "inauspiciousTimes": {
    "rahuKaal": {
      "start": "2026-01-07T12:26:00+05:30",
      "end": "2026-01-07T13:48:00+05:30"
    },
    "yamaganda": {
      "start": "2026-01-07T08:36:00+05:30",
      "end": "2026-01-07T09:58:00+05:30"
    },
    "gulikaKaal": {
      "start": "2026-01-07T11:04:00+05:30",
      "end": "2026-01-07T12:26:00+05:30"
    }
  }
}
```

## Get Individual Elements

```
// Get only Tithi
const tithi = await vedika.tithi({
  date: '2026-01-07',
  latitude: 28.6139,
  longitude: 77.2090
});

// Get only Nakshatra
const nakshatra = await vedika.nakshatra({
  date: '2026-01-07',
  latitude: 28.6139,
  longitude: 77.2090
});

// Get Rahu Kaal
const rahuKaal = await vedika.rahuKaal({
  date: '2026-01-07',
  latitude: 28.6139,
  longitude: 77.2090
});

// Get Choghadiya (hourly auspiciousness)
const choghadiya = await vedika.choghadiya({
  date: '2026-01-07',
  latitude: 28.6139,
  longitude: 77.2090
});
```

## Choghadiya Response

```
{
  "day": [
    {
      "name": "Udveg",
      "type": "inauspicious",
      "lord": "Sun",
      "start": "07:14:00",
      "end": "08:36:00"
    },
    {
      "name": "Char",
      "type": "good",
      "lord": "Venus",
      "start": "08:36:00",
      "end": "09:58:00"
    },
    {
      "name": "Labh",
      "type": "auspicious",
      "lord": "Mercury",
      "start": "09:58:00",
      "end": "11:20:00"
    },
    // ... continues for day hours
  ],
  "night": [
    {
      "name": "Rog",
      "type": "inauspicious",
      "lord": "Mars",
      "start": "17:38:00",
      "end": "19:17:00"
    },
    // ... continues for night hours
  ]
}
```

## Panchang for Date Range

```
// Get panchang for a week
const weekPanchang = await vedika.panchangRange({
  startDate: '2026-01-07',
  endDate: '2026-01-14',
  latitude: 28.6139,
  longitude: 77.2090,
  timezone: 'Asia/Kolkata'
});

weekPanchang.forEach(day => {
  console.log(`${day.date}: ${day.tithi.name} - ${day.nakshatra.name}`);
});

// Find auspicious days in range
const auspiciousDays = weekPanchang.filter(day =>
  day.tithi.isAuspicious && day.yoga.isAuspicious
);

console.log('Auspicious days:', auspiciousDays.map(d => d.date));
```

## Festival Detection

```
// Get panchang with festivals
const panchang = await vedika.panchang({
  date: '2026-03-14',
  latitude: 28.6139,
  longitude: 77.2090,
  includeFestivals: true
});

if (panchang.festivals.length > 0) {
  console.log('Festivals today:');
  panchang.festivals.forEach(f => {
    console.log(`- ${f.name}: ${f.description}`);
  });
}

// Output:
// Festivals today:
// - Holi: Festival of Colors
```

## Build a Calendar Widget

```
async function buildPanchangCalendar(year, month, location) {
  const vedika = new VedikaClient();

  const daysInMonth = new Date(year, month, 0).getDate();
  const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
  const endDate = `${year}-${String(month).padStart(2, '0')}-${daysInMonth}`;

  const monthData = await vedika.panchangRange({
    startDate,
    endDate,
    ...location
  });

  // Transform for calendar display
  return monthData.map(day => ({
    date: day.date,
    tithi: day.tithi.name,
    nakshatra: day.nakshatra.name,
    festivals: day.festivals || [],
    isAuspicious: day.tithi.isAuspicious && !hasInauspiciousYoga(day),
    rahuKaal: `${day.inauspiciousTimes.rahuKaal.start} - ${day.inauspiciousTimes.rahuKaal.end}`
  }));
}

function hasInauspiciousYoga(day) {
  const inauspiciousYogas = ['Vishkumbha', 'Atiganda', 'Vyaghata'];
  return inauspiciousYogas.includes(day.yoga.name);
}

// Usage
const calendar = await buildPanchangCalendar(2026, 1, {
  latitude: 28.6139,
  longitude: 77.2090,
  timezone: 'Asia/Kolkata'
});

console.log(calendar);
```

## Related APIs

[ **

#### Panchang Widget

Build calendar widgets

](https://vedika.io/docs/sdk/panchang-widget) [ **

#### Dasha Periods

Planetary periods

](https://vedika.io/docs/sdk/dasha) [ **

#### Choghadiya API

Hourly muhurtas

](https://vedika.io/docs/api/v2/astrology/choghadiya)

---
_Full page (always current): [https://vedika.io/docs/sdk/panchang](https://vedika.io/docs/sdk/panchang) · [Vedika docs](https://vedika.io/docs)_
