---
title: "Shopify Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/shopify-integration
canonical: https://vedika.io/docs/sdk/shopify-integration
---

# Shopify Integration

Astrology-powered e-commerce

## Use Cases

#### Gemstone Recommendations

Suggest gemstones based on birth chart

#### Lucky Products

Products matched to zodiac signs

#### Auspicious Purchase Days

Show best days to buy based on panchang

#### Personalized Gifts

Gift suggestions by recipient's sign

## Shopify App Setup

```
// server.js - Shopify App Backend
import express from 'express';
import { VedikaClient } from '@vedika-io/sdk';

const app = express();
const vedika = new VedikaClient();

// Gemstone recommendations based on birth chart
app.post('/api/gemstone-recommendations', async (req, res) => {
  const { birthDate, birthTime, latitude, longitude } = req.body;

  const chart = await vedika.birthChart({
    datetime: `${birthDate}T${birthTime}:00`,
    latitude,
    longitude
  });

  // Map planets to gemstones
  const gemstoneMap = {
    Sun: { gemstone: 'Ruby', metal: 'Gold' },
    Moon: { gemstone: 'Pearl', metal: 'Silver' },
    Mars: { gemstone: 'Red Coral', metal: 'Gold' },
    Mercury: { gemstone: 'Emerald', metal: 'Gold' },
    Jupiter: { gemstone: 'Yellow Sapphire', metal: 'Gold' },
    Venus: { gemstone: 'Diamond', metal: 'Platinum' },
    Saturn: { gemstone: 'Blue Sapphire', metal: 'Silver' },
    Rahu: { gemstone: 'Hessonite', metal: 'Silver' },
    Ketu: { gemstone: 'Cat\'s Eye', metal: 'Silver' }
  };

  // Find weak planets that need strengthening
  const recommendations = chart.planets
    .filter(p => p.isDebilitated || p.isCombust)
    .map(p => ({
      planet: p.name,
      reason: p.isDebilitated ? 'Debilitated' : 'Combust',
      ...gemstoneMap[p.name]
    }));

  // Also recommend based on ascendant lord
  const ascendantLord = chart.houses[0].signLord;
  recommendations.unshift({
    planet: ascendantLord,
    reason: 'Ascendant Lord - Primary Recommendation',
    ...gemstoneMap[ascendantLord]
  });

  res.json({ recommendations });
});
```

## Theme Liquid Snippet

```
<!-- snippets/vedika-zodiac-selector.liquid -->
<div class="vedika-zodiac-selector">
  <h3>Shop by Zodiac Sign</h3>
  <div class="zodiac-grid">
    {% assign signs = "aries,taurus,gemini,cancer,leo,virgo,libra,scorpio,sagittarius,capricorn,aquarius,pisces" | split: "," %}
    {% for sign in signs %}
      <a href="/collections/{{ sign }}" class="zodiac-item">
        <img src="{{ sign | append: '.svg' | asset_url }}" alt="{{ sign }}">
        <span>{{ sign | capitalize }}</span>
      </a>
    {% endfor %}
  </div>
</div>

<!-- snippets/vedika-birth-chart-form.liquid -->
<div id="vedika-chart-widget">
  <h3>Find Your Lucky Gemstone</h3>
  <form id="birth-details-form">
    <input type="date" name="birthDate" required>
    <input type="time" name="birthTime" required>
    <input type="text" name="birthPlace" placeholder="Birth City" required>
    <button type="submit">Get Recommendations</button>
  </form>
  <div id="recommendations"></div>
</div>

<script>
document.getElementById('birth-details-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);

  // Geocode birth place (use a geocoding service)
  const coords = await geocodePlace(formData.get('birthPlace'));

  const response = await fetch('/apps/vedika/gemstone-recommendations', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      birthDate: formData.get('birthDate'),
      birthTime: formData.get('birthTime'),
      latitude: coords.lat,
      longitude: coords.lng
    })
  });

  const data = await response.json();
  displayRecommendations(data.recommendations);
});
</script>
```

## Product Metafields

```
// Tag products with astrological attributes
// In Shopify Admin > Products > Edit > Metafields

// Example metafields for a Ruby gemstone:
{
  "vedika": {
    "ruling_planet": "Sun",
    "zodiac_signs": ["Leo"],
    "chakra": "Heart",
    "benefits": ["Leadership", "Confidence", "Vitality"],
    "day_to_wear": "Sunday",
    "finger": "Ring finger",
    "metal": "Gold"
  }
}
```

## Smart Collections

```
<!-- sections/zodiac-products.liquid -->
{% assign customer_zodiac = customer.metafields.vedika.zodiac_sign %}

{% if customer_zodiac %}
  <h2>Products for {{ customer_zodiac | capitalize }}</h2>
  {% assign zodiac_collection = collections[customer_zodiac] %}
  {% for product in zodiac_collection.products limit: 8 %}
    {% render 'product-card', product: product %}
  {% endfor %}
{% else %}
  <!-- Show zodiac selector -->
  {% render 'vedika-zodiac-selector' %}
{% endif %}
```

## Auspicious Day Widget

```
<!-- Show auspicious times for purchase -->
<div id="vedika-muhurta">
  <h4>Today's Auspicious Times for Purchase</h4>
  <div id="muhurta-times">Loading...</div>
</div>

<script>
async function loadMuhurta() {
  const response = await fetch('/apps/vedika/choghadiya');
  const data = await response.json();

  const auspicious = data.filter(slot => slot.type === 'auspicious');

  document.getElementById('muhurta-times').innerHTML = auspicious
    .map(slot => `
      <div class="muhurta-slot ${slot.isActive ? 'active' : ''}">
        <span class="time">${slot.start} - ${slot.end}</span>
        <span class="name">${slot.name}</span>
      </div>
    `)
    .join('');
}

loadMuhurta();
</script>
```

## Next Steps

[ **

#### Kundli Generator

Build chart generators

](https://vedika.io/docs/sdk/kundli-generator) [ **

#### Mobile Apps

React Native / Flutter

](https://vedika.io/docs/sdk/mobile-app)

---
_Full page (always current): [https://vedika.io/docs/sdk/shopify-integration](https://vedika.io/docs/sdk/shopify-integration) · [Vedika docs](https://vedika.io/docs)_
