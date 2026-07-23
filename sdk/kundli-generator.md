---
title: "Build a Kundli Generator | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/kundli-generator
canonical: https://vedika.io/docs/sdk/kundli-generator
---

# Build a Kundli Generator

Professional birth chart visualization

## Chart Types

#### North Indian

Diamond-shaped chart

#### South Indian

Square grid chart

#### Western

Circular wheel chart

## Birth Details Form (React)

```
// components/BirthDetailsForm.tsx
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { PlaceAutocomplete } from './PlaceAutocomplete';

const birthSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, 'Invalid date'),
  time: z.string().regex(/^\d{2}:\d{2}$/, 'Invalid time'),
  place: z.string().min(1, 'Birth place is required'),
  latitude: z.number().min(-90).max(90),
  longitude: z.number().min(-180).max(180),
  timezone: z.string()
});

type BirthDetails = z.infer<typeof birthSchema>;

interface Props {
  onSubmit: (details: BirthDetails) => void;
  loading?: boolean;
}

export function BirthDetailsForm({ onSubmit, loading }: Props) {
  const { register, handleSubmit, setValue, formState: { errors } } = useForm<BirthDetails>({
    resolver: zodResolver(birthSchema)
  });

  const [selectedPlace, setSelectedPlace] = useState<string>('');

  function handlePlaceSelect(place: {
    name: string;
    lat: number;
    lng: number;
    timezone: string
  }) {
    setSelectedPlace(place.name);
    setValue('place', place.name);
    setValue('latitude', place.lat);
    setValue('longitude', place.lng);
    setValue('timezone', place.timezone);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Full Name</label>
        <input
          {...register('name')}
          className="w-full border rounded-lg px-3 py-2"
          placeholder="Enter name"
        />
        {errors.name && <p className="text-red-500 text-sm">{errors.name.message}</p>}
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium mb-1">Birth Date</label>
          <input
            {...register('date')}
            type="date"
            className="w-full border rounded-lg px-3 py-2"
          />
          {errors.date && <p className="text-red-500 text-sm">{errors.date.message}</p>}
        </div>
        <div>
          <label className="block text-sm font-medium mb-1">Birth Time</label>
          <input
            {...register('time')}
            type="time"
            className="w-full border rounded-lg px-3 py-2"
          />
          {errors.time && <p className="text-red-500 text-sm">{errors.time.message}</p>}
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Birth Place</label>
        <PlaceAutocomplete
          value={selectedPlace}
          onChange={handlePlaceSelect}
          placeholder="Search city..."
        />
        {errors.place && <p className="text-red-500 text-sm">{errors.place.message}</p>}
      </div>

      <input type="hidden" {...register('latitude')} />
      <input type="hidden" {...register('longitude')} />
      <input type="hidden" {...register('timezone')} />

      <button
        type="submit"
        disabled={loading}
        className="w-full bg-indigo-600 text-white py-3 rounded-lg hover:bg-indigo-700 disabled:opacity-50"
      >
        {loading ? 'Generating Chart...' : 'Generate Kundli'}
      </button>
    </form>
  );
}
```

## Vedika API Integration

```
// services/kundli.ts
import { VedikaClient } from '@vedika-io/sdk';

const vedika = new VedikaClient();

interface BirthDetails {
  datetime: string;
  latitude: number;
  longitude: number;
  timezone?: string;
}

interface KundliData {
  chart: any;
  planets: Planet[];
  houses: House[];
  dasha: DashaPeriod[];
  yogas: Yoga[];
  doshas: Dosha[];
  divisionalCharts: Record<string, any>;
}

export async function generateKundli(birthDetails: BirthDetails): Promise<KundliData> {
  // Fetch all data in parallel for performance
  const [chart, dasha, mangalDosha, kaalSarpDosha] = await Promise.all([
    vedika.birthChart(birthDetails),
    vedika.vimshottariDasha(birthDetails),
    vedika.mangalDosha(birthDetails),
    vedika.kaalSarpDosha(birthDetails)
  ]);

  // Get divisional charts
  const divisionalCharts: Record<string, any> = {};
  const divisions = ['D1', 'D2', 'D3', 'D4', 'D7', 'D9', 'D10', 'D12'];

  for (const division of divisions) {
    divisionalCharts[division] = await vedika.divisionalChart({
      ...birthDetails,
      division
    });
  }

  // Combine dosha results
  const doshas: Dosha[] = [];
  if (mangalDosha.hasDosha) {
    doshas.push({
      name: 'Mangal Dosha',
      severity: mangalDosha.severity,
      details: mangalDosha.details,
      remedies: mangalDosha.remedies
    });
  }
  if (kaalSarpDosha.hasDosha) {
    doshas.push({
      name: 'Kaal Sarp Dosha',
      type: kaalSarpDosha.type,
      details: kaalSarpDosha.details,
      remedies: kaalSarpDosha.remedies
    });
  }

  return {
    chart,
    planets: chart.planets,
    houses: chart.houses,
    dasha: dasha.periods,
    yogas: chart.yogas || [],
    doshas,
    divisionalCharts
  };
}
```

## North Indian Chart (SVG)

```
// components/NorthIndianChart.tsx
import React from 'react';

interface Planet {
  name: string;
  symbol: string;
  house: number;
  degree: number;
  isRetrograde: boolean;
}

interface Props {
  planets: Planet[];
  ascendant: number;
  size?: number;
}

// House positions in North Indian chart (diamond layout)
const HOUSE_POSITIONS = [
  { x: 150, y: 0 },    // 1 - top center
  { x: 75, y: 37.5 },  // 2 - upper left
  { x: 0, y: 75 },     // 3 - left upper
  { x: 0, y: 150 },    // 4 - left lower
  { x: 75, y: 225 },   // 5 - lower left
  { x: 150, y: 262.5 },// 6 - bottom center
  { x: 225, y: 225 },  // 7 - lower right
  { x: 300, y: 150 },  // 8 - right lower
  { x: 300, y: 75 },   // 9 - right upper
  { x: 225, y: 37.5 }, // 10 - upper right
  { x: 225, y: 112.5 },// 11 - right middle
  { x: 75, y: 112.5 }  // 12 - left middle
];

const PLANET_SYMBOLS: Record<string, string> = {
  Sun: 'Su',
  Moon: 'Mo',
  Mars: 'Ma',
  Mercury: 'Me',
  Jupiter: 'Ju',
  Venus: 'Ve',
  Saturn: 'Sa',
  Rahu: 'Ra',
  Ketu: 'Ke'
};

export function NorthIndianChart({ planets, ascendant, size = 300 }: Props) {
  const scale = size / 300;

  // Group planets by house
  const planetsByHouse: Record<number, Planet[]> = {};
  planets.forEach(planet => {
    const house = planet.house;
    if (!planetsByHouse[house]) planetsByHouse[house] = [];
    planetsByHouse[house].push(planet);
  });

  return (
    <svg
      width={size}
      height={size}
      viewBox="0 0 300 300"
      className="kundli-chart"
    >
      {/* Background */}
      <rect width="300" height="300" fill="#FFF9E6" />

      {/* Outer square */}
      <rect
        x="10" y="10"
        width="280" height="280"
        fill="none"
        stroke="#8B4513"
        strokeWidth="2"
      />

      {/* Diagonal lines creating diamond pattern */}
      <line x1="10" y1="10" x2="150" y2="150" stroke="#8B4513" strokeWidth="1" />
      <line x1="290" y1="10" x2="150" y2="150" stroke="#8B4513" strokeWidth="1" />
      <line x1="10" y1="290" x2="150" y2="150" stroke="#8B4513" strokeWidth="1" />
      <line x1="290" y1="290" x2="150" y2="150" stroke="#8B4513" strokeWidth="1" />

      {/* Inner diamond */}
      <polygon
        points="150,10 290,150 150,290 10,150"
        fill="none"
        stroke="#8B4513"
        strokeWidth="1"
      />

      {/* House numbers */}
      {[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12].map(house => {
        const pos = getHouseNumberPosition(house);
        return (
          <text
            key={house}
            x={pos.x}
            y={pos.y}
            fontSize="10"
            fill="#999"
            textAnchor="middle"
          >
            {house}
          </text>
        );
      })}

      {/* Ascendant marker */}
      <text x="150" y="25" fontSize="12" fill="#C41E3A" textAnchor="middle" fontWeight="bold">
        Asc
      </text>

      {/* Planets */}
      {Object.entries(planetsByHouse).map(([house, housePlanets]) => {
        const basePos = HOUSE_POSITIONS[parseInt(house) - 1];
        return housePlanets.map((planet, index) => {
          const offsetY = index * 14;
          const symbol = PLANET_SYMBOLS[planet.name] || planet.name.slice(0, 2);

          return (
            <g key={planet.name}>
              <text
                x={basePos.x + 25}
                y={basePos.y + 20 + offsetY}
                fontSize="11"
                fill={getPlanetColor(planet.name)}
                fontWeight="bold"
              >
                {symbol}
                {planet.isRetrograde && <tspan fontSize="8"> R</tspan>}
              </text>
              <text
                x={basePos.x + 50}
                y={basePos.y + 20 + offsetY}
                fontSize="9"
                fill="#666"
              >
                {planet.degree.toFixed(1)}°
              </text>
            </g>
          );
        });
      })}
    </svg>
  );
}

function getPlanetColor(planet: string): string {
  const colors: Record<string, string> = {
    Sun: '#FF6B00',
    Moon: '#4169E1',
    Mars: '#DC143C',
    Mercury: '#228B22',
    Jupiter: '#FFD700',
    Venus: '#FF69B4',
    Saturn: '#4B0082',
    Rahu: '#2F4F4F',
    Ketu: '#8B4513'
  };
  return colors[planet] || '#333';
}

function getHouseNumberPosition(house: number): { x: number; y: number } {
  // Positions for house numbers (small, in corner of each house)
  const positions: Record<number, { x: number; y: number }> = {
    1: { x: 150, y: 45 },
    2: { x: 80, y: 55 },
    3: { x: 25, y: 100 },
    4: { x: 25, y: 200 },
    5: { x: 80, y: 250 },
    6: { x: 150, y: 280 },
    7: { x: 220, y: 250 },
    8: { x: 275, y: 200 },
    9: { x: 275, y: 100 },
    10: { x: 220, y: 55 },
    11: { x: 220, y: 150 },
    12: { x: 80, y: 150 }
  };
  return positions[house];
}
```

## South Indian Chart (SVG)

```
// components/SouthIndianChart.tsx
import React from 'react';

interface Props {
  planets: Planet[];
  ascendant: number;
  size?: number;
}

// South Indian chart: fixed positions, signs don't move
const SIGN_POSITIONS = [
  { row: 0, col: 1 }, // Pisces
  { row: 0, col: 0 }, // Aries
  { row: 1, col: 0 }, // Taurus
  { row: 2, col: 0 }, // Gemini
  { row: 3, col: 0 }, // Cancer
  { row: 3, col: 1 }, // Leo
  { row: 3, col: 2 }, // Virgo
  { row: 3, col: 3 }, // Libra
  { row: 2, col: 3 }, // Scorpio
  { row: 1, col: 3 }, // Sagittarius
  { row: 0, col: 3 }, // Capricorn
  { row: 0, col: 2 }  // Aquarius
];

const SIGN_NAMES = [
  'Aries', 'Taurus', 'Gemini', 'Cancer',
  'Leo', 'Virgo', 'Libra', 'Scorpio',
  'Sagittarius', 'Capricorn', 'Aquarius', 'Pisces'
];

export function SouthIndianChart({ planets, ascendant, size = 300 }: Props) {
  const cellSize = size / 4;

  // Group planets by sign (1-12)
  const planetsBySign: Record<number, Planet[]> = {};
  planets.forEach(planet => {
    const sign = planet.sign;
    if (!planetsBySign[sign]) planetsBySign[sign] = [];
    planetsBySign[sign].push(planet);
  });

  return (
    <svg width={size} height={size} viewBox={`0 0 ${size} ${size}`}>
      {/* Background */}
      <rect width={size} height={size} fill="#F0F8FF" />

      {/* Grid lines */}
      {[0, 1, 2, 3, 4].map(i => (
        <React.Fragment key={i}>
          <line
            x1={i * cellSize} y1="0"
            x2={i * cellSize} y2={size}
            stroke="#4682B4" strokeWidth="1"
          />
          <line
            x1="0" y1={i * cellSize}
            x2={size} y2={i * cellSize}
            stroke="#4682B4" strokeWidth="1"
          />
        </React.Fragment>
      ))}

      {/* Center cells (not used in South Indian) */}
      <rect
        x={cellSize} y={cellSize}
        width={cellSize * 2} height={cellSize * 2}
        fill="#E6F3FF"
      />

      {/* Sign labels and planets */}
      {SIGN_POSITIONS.map((pos, signIndex) => {
        const x = pos.col * cellSize;
        const y = pos.row * cellSize;
        const signNumber = signIndex + 1;
        const isAscendant = signNumber === ascendant;

        return (
          <g key={signIndex}>
            {/* Highlight ascendant */}
            {isAscendant && (
              <rect
                x={x + 2} y={y + 2}
                width={cellSize - 4} height={cellSize - 4}
                fill="#FFE4B5" stroke="#DAA520" strokeWidth="2"
              />
            )}

            {/* Sign abbreviation */}
            <text
              x={x + 5}
              y={y + 12}
              fontSize="9"
              fill="#666"
            >
              {SIGN_NAMES[signIndex].slice(0, 3)}
            </text>

            {/* Ascendant marker */}
            {isAscendant && (
              <text
                x={x + cellSize - 15}
                y={y + 12}
                fontSize="8"
                fill="#C41E3A"
                fontWeight="bold"
              >
                Asc
              </text>
            )}

            {/* Planets in this sign */}
            {(planetsBySign[signNumber] || []).map((planet, idx) => (
              <text
                key={planet.name}
                x={x + 5 + (idx % 2) * 35}
                y={y + 28 + Math.floor(idx / 2) * 14}
                fontSize="10"
                fill={getPlanetColor(planet.name)}
                fontWeight="bold"
              >
                {PLANET_SYMBOLS[planet.name]}
                {planet.isRetrograde && 'ᴿ'}
              </text>
            ))}
          </g>
        );
      })}

      {/* Center text */}
      <text
        x={size / 2}
        y={size / 2}
        textAnchor="middle"
        fontSize="12"
        fill="#4682B4"
      >
        Rashi Chart
      </text>
    </svg>
  );
}
```

## Planet Details Component

```
// components/PlanetDetails.tsx
import React from 'react';

interface Planet {
  name: string;
  sign: string;
  signLord: string;
  house: number;
  degree: number;
  nakshatra: string;
  nakshatraLord: string;
  isRetrograde: boolean;
  isExalted: boolean;
  isDebilitated: boolean;
  isCombust: boolean;
}

interface Props {
  planets: Planet[];
}

export function PlanetDetails({ planets }: Props) {
  return (
    <div className="overflow-x-auto">
      <table className="w-full border-collapse">
        <thead>
          <tr className="bg-indigo-50">
            <th className="border p-2 text-left">Planet</th>
            <th className="border p-2 text-left">Sign</th>
            <th className="border p-2 text-center">House</th>
            <th className="border p-2 text-center">Degree</th>
            <th className="border p-2 text-left">Nakshatra</th>
            <th className="border p-2 text-center">Status</th>
          </tr>
        </thead>
        <tbody>
          {planets.map(planet => (
            <tr key={planet.name} className="hover:bg-gray-50">
              <td className="border p-2">
                <span
                  className="font-semibold"
                  style={{ color: getPlanetColor(planet.name) }}
                >
                  {planet.name}
                </span>
              </td>
              <td className="border p-2">
                {planet.sign}
                <span className="text-gray-500 text-sm ml-1">
                  ({planet.signLord})
                </span>
              </td>
              <td className="border p-2 text-center">{planet.house}</td>
              <td className="border p-2 text-center">
                {planet.degree.toFixed(2)}°
              </td>
              <td className="border p-2">
                {planet.nakshatra}
                <span className="text-gray-500 text-sm ml-1">
                  ({planet.nakshatraLord})
                </span>
              </td>
              <td className="border p-2 text-center">
                <div className="flex gap-1 justify-center flex-wrap">
                  {planet.isRetrograde && (
                    <span className="bg-orange-100 text-orange-700 px-1.5 py-0.5 rounded text-xs">
                      Retro
                    </span>
                  )}
                  {planet.isExalted && (
                    <span className="bg-green-100 text-green-700 px-1.5 py-0.5 rounded text-xs">
                      Exalted
                    </span>
                  )}
                  {planet.isDebilitated && (
                    <span className="bg-red-100 text-red-700 px-1.5 py-0.5 rounded text-xs">
                      Debil
                    </span>
                  )}
                  {planet.isCombust && (
                    <span className="bg-yellow-100 text-yellow-700 px-1.5 py-0.5 rounded text-xs">
                      Combust
                    </span>
                  )}
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## Dasha Timeline Component

```
// components/DashaTimeline.tsx
import React, { useState } from 'react';

interface DashaPeriod {
  planet: string;
  startDate: string;
  endDate: string;
  antardashas: AntardashaPeriod[];
}

interface AntardashaPeriod {
  planet: string;
  startDate: string;
  endDate: string;
}

interface Props {
  periods: DashaPeriod[];
  currentDate?: Date;
}

export function DashaTimeline({ periods, currentDate = new Date() }: Props) {
  const [expanded, setExpanded] = useState<string | null>(null);

  // Find current period
  const currentPeriod = periods.find(p => {
    const start = new Date(p.startDate);
    const end = new Date(p.endDate);
    return currentDate >= start && currentDate <= end;
  });

  return (
    <div className="space-y-2">
      {periods.map(period => {
        const isCurrent = period === currentPeriod;
        const isExpanded = expanded === period.planet;
        const startYear = new Date(period.startDate).getFullYear();
        const endYear = new Date(period.endDate).getFullYear();

        return (
          <div key={period.planet} className="border rounded-lg overflow-hidden">
            <button
              onClick={() => setExpanded(isExpanded ? null : period.planet)}
              className={`w-full p-3 flex items-center justify-between ${
                isCurrent ? 'bg-indigo-50 border-l-4 border-indigo-500' : 'bg-gray-50'
              }`}
            >
              <div className="flex items-center gap-3">
                <div
                  className="w-8 h-8 rounded-full flex items-center justify-center text-white font-bold"
                  style={{ backgroundColor: getPlanetColor(period.planet) }}
                >
                  {period.planet.slice(0, 2)}
                </div>
                <div className="text-left">
                  <div className="font-semibold">
                    {period.planet} Mahadasha
                    {isCurrent && (
                      <span className="ml-2 text-xs bg-indigo-500 text-white px-2 py-0.5 rounded">
                        Current
                      </span>
                    )}
                  </div>
                  <div className="text-sm text-gray-500">
                    {startYear} - {endYear}
                  </div>
                </div>
              </div>
              <i className={`fas fa-chevron-${isExpanded ? 'up' : 'down'} text-gray-400`} />
            </button>

            {isExpanded && (
              <div className="p-3 bg-white border-t">
                <h4 className="text-sm font-semibold text-gray-600 mb-2">Antardashas</h4>
                <div className="grid grid-cols-3 gap-2">
                  {period.antardashas.map(ad => {
                    const adStart = new Date(ad.startDate);
                    const adEnd = new Date(ad.endDate);
                    const isCurrentAd = currentDate >= adStart && currentDate <= adEnd;

                    return (
                      <div
                        key={ad.planet}
                        className={`p-2 rounded text-sm ${
                          isCurrentAd ? 'bg-indigo-100 border border-indigo-300' : 'bg-gray-100'
                        }`}
                      >
                        <div className="font-medium">{ad.planet}</div>
                        <div className="text-xs text-gray-500">
                          {formatDate(adStart)} - {formatDate(adEnd)}
                        </div>
                      </div>
                    );
                  })}
                </div>
              </div>
            )}
          </div>
        );
      })}
    </div>
  );
}

function formatDate(date: Date): string {
  return date.toLocaleDateString('en-IN', {
    month: 'short',
    year: 'numeric'
  });
}
```

## PDF Export

```
// services/pdfExport.ts
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

export async function exportKundliPDF(
  kundliData: KundliData,
  chartElement: HTMLElement
): Promise<Blob> {
  const pdf = new jsPDF('p', 'mm', 'a4');
  const pageWidth = pdf.internal.pageSize.getWidth();

  // Title
  pdf.setFontSize(20);
  pdf.setTextColor(75, 0, 130);
  pdf.text('Vedic Birth Chart (Kundli)', pageWidth / 2, 20, { align: 'center' });

  // Birth details
  pdf.setFontSize(12);
  pdf.setTextColor(0, 0, 0);
  pdf.text(`Name: ${kundliData.name}`, 20, 35);
  pdf.text(`Date: ${kundliData.birthDate}`, 20, 42);
  pdf.text(`Time: ${kundliData.birthTime}`, 20, 49);
  pdf.text(`Place: ${kundliData.birthPlace}`, 20, 56);

  // Chart image
  const canvas = await html2canvas(chartElement, { scale: 2 });
  const imgData = canvas.toDataURL('image/png');
  pdf.addImage(imgData, 'PNG', 20, 65, 80, 80);

  // Planet positions table
  pdf.setFontSize(14);
  pdf.text('Planetary Positions', 110, 70);

  pdf.setFontSize(10);
  let y = 80;
  kundliData.planets.forEach(planet => {
    pdf.text(`${planet.name}: ${planet.sign} (${planet.degree.toFixed(2)}°)`, 110, y);
    y += 6;
  });

  // Dasha periods
  pdf.addPage();
  pdf.setFontSize(16);
  pdf.text('Vimshottari Dasha Periods', 20, 20);

  pdf.setFontSize(10);
  y = 35;
  kundliData.dasha.slice(0, 5).forEach(period => {
    pdf.setFontSize(12);
    pdf.text(`${period.planet} Mahadasha`, 20, y);
    pdf.setFontSize(9);
    pdf.text(`${period.startDate} to ${period.endDate}`, 20, y + 5);
    y += 15;
  });

  // Doshas
  if (kundliData.doshas.length > 0) {
    pdf.addPage();
    pdf.setFontSize(16);
    pdf.text('Dosha Analysis', 20, 20);

    y = 35;
    kundliData.doshas.forEach(dosha => {
      pdf.setFontSize(12);
      pdf.setTextColor(220, 20, 60);
      pdf.text(dosha.name, 20, y);
      pdf.setTextColor(0, 0, 0);
      pdf.setFontSize(10);
      pdf.text(dosha.details, 20, y + 6, { maxWidth: 170 });
      y += 25;
    });
  }

  return pdf.output('blob');
}

// Usage
async function downloadPDF() {
  const chartElement = document.getElementById('kundli-chart');
  const blob = await exportKundliPDF(kundliData, chartElement);

  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `kundli-${kundliData.name}.pdf`;
  a.click();
  URL.revokeObjectURL(url);
}
```

## Next Steps

[ **

#### Matching Portal

Kundli matching for matrimonial

](https://vedika.io/docs/sdk/matching-portal) [ **

#### Dasha API

Detailed Dasha calculations

](https://vedika.io/docs/sdk/dasha)

---
_Full page (always current): [https://vedika.io/docs/sdk/kundli-generator](https://vedika.io/docs/sdk/kundli-generator) · [Vedika docs](https://vedika.io/docs)_
