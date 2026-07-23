---
title: "Vedic Astrology Calculations Guide | Technical Deep-Dive"
source: https://vedika.io/guides/vedic-astrology-calculations-guide
canonical: https://vedika.io/guides/vedic-astrology-calculations-guide
---

4000+ Words • Technical Deep-Dive

# Vedic Astrology Calculations Guide

How Ayanamsa, Dashas, Yogas, and Doshas are Calculated

Complete technical explanation of Vedic astrology mathematics for developers and astrologers

## Understanding Ayanamsa

Ayanamsa is the angular difference between the tropical and sidereal zodiacs. This correction is fundamental to Vedic astrology calculations.

### Why Ayanamsa Matters

Western astrology uses the **tropical zodiac** which is fixed to the seasons (Aries starts at the spring equinox). Vedic astrology uses the **sidereal zodiac** which is fixed to the actual star constellations.

Due to Earth's **precession** (wobbling on its axis), the spring equinox slowly drifts backward through the zodiac at a rate of approximately 50.3 arc-seconds per year. This creates an ever-increasing gap between the two zodiacs called the Ayanamsa.

### Major Ayanamsa Systems

| System | Reference Point | Value (2025) | Usage |
| --- | --- | --- | --- |
| **Lahiri (Chitrapaksha)** | Spica at 180° in 285 CE | 24°10' | Most popular in India, official |
| **Raman** | Based on B.V. Raman's research | 22°23' | Popular in South India |
| **Krishnamurti (KP)** | Custom calculations | 23°53' | KP system astrology |
| **Yukteshwar** | Based on Sri Yukteshwar's work | 20°55' | Spiritual astrology |
| **Djwhal Khul** | Esoteric astrology | 24°02' | Western esoteric |

### Calculation Formula

```
def calculate_lahiri_ayanamsa(julian_day):
    """Calculate Lahiri Ayanamsa for given Julian Day"""
    # Base calculation
    t = (julian_day - 2451545.0) / 36525  # Julian centuries from J2000

    # Ayanamsa formula (simplified)
    ayanamsa = 23.85 + (t * 0.0139)  # degrees

    # Precession rate
    precession_rate = 50.27 / 3600  # arc-seconds to degrees per year

    return ayanamsa

# Convert tropical longitude to sidereal
def tropical_to_sidereal(tropical_long, ayanamsa):
    sidereal = tropical_long - ayanamsa
    if sidereal < 0:
        sidereal += 360
    return sidereal

```

### Impact on Chart Interpretation

The Ayanamsa difference (currently ~24°) means:
- Most Western "Aries" Suns become Vedic "Pisces" Suns
- Ascendant signs often shift by one sign
- House cusps change significantly
- Dasha periods are calculated differently

## Planetary Position Calculations

### XALEN Ephemeris Integration

The gold standard for astronomical calculations is the XALEN Ephemeris, which provides accuracy to 0.001 degrees.
```
import ephemeris as ephe

def calculate_planetary_positions(jd, lat, lon, ayanamsa='LAHIRI'):
    """Calculate all planetary positions for given time and location"""

    # Set Ayanamsa
    ephe.set_sid_mode(ephe.SIDM_LAHIRI)

    planets = {
        'Sun': ephe.SUN,
        'Moon': ephe.MOON,
        'Mars': ephe.MARS,
        'Mercury': ephe.MERCURY,
        'Jupiter': ephe.JUPITER,
        'Venus': ephe.VENUS,
        'Saturn': ephe.SATURN,
        'Rahu': ephe.TRUE_NODE,
        'Ketu': ephe.TRUE_NODE  # Ketu = Rahu + 180°
    }

    positions = {}
    for planet_name, planet_id in planets.items():
        # Calculate tropical position
        result = ephe.calc_ut(jd, planet_id)
        tropical_long = result[0][0]

        # Get ayanamsa value
        ayanamsa_val = ephe.get_ayanamsa_ut(jd)

        # Convert to sidereal
        sidereal_long = tropical_long - ayanamsa_val
        if sidereal_long < 0:
            sidereal_long += 360

        # Handle Ketu (opposite of Rahu)
        if planet_name == 'Ketu':
            sidereal_long = (sidereal_long + 180) % 360

        # Determine sign and degree
        sign_num = int(sidereal_long / 30)
        degree = sidereal_long % 30

        positions[planet_name] = {
            'longitude': sidereal_long,
            'sign': get_sign_name(sign_num),
            'degree': degree,
            'speed': result[0][3],  # Daily motion
            'latitude': result[0][1],  # Celestial latitude
            'retrograde': result[0][3] < 0
        }

    return positions

```

### Nakshatra Calculation

Nakshatras are 27 lunar mansions, each spanning 13°20' (360° / 27). The nakshatra and pada are critical for Dasha calculations.
```
def calculate_nakshatra(longitude):
    """Calculate nakshatra and pada from sidereal longitude"""

    # Each nakshatra is 13.333... degrees
    nakshatra_span = 360 / 27
    pada_span = nakshatra_span / 4

    # Calculate nakshatra number (0-26)
    nakshatra_num = int(longitude / nakshatra_span)

    # Calculate pada (1-4)
    nakshatra_start = nakshatra_num * nakshatra_span
    pada_num = int((longitude - nakshatra_start) / pada_span) + 1

    # Degree within nakshatra
    nak_degree = longitude - nakshatra_start

    nakshatras = [
        'Ashwini', 'Bharani', 'Krittika', 'Rohini', 'Mrigashira', 'Ardra',
        'Punarvasu', 'Pushya', 'Ashlesha', 'Magha', 'Purva Phalguni',
        'Uttara Phalguni', 'Hasta', 'Chitra', 'Swati', 'Vishakha',
        'Anuradha', 'Jyeshtha', 'Mula', 'Purva Ashadha', 'Uttara Ashadha',
        'Shravana', 'Dhanishta', 'Shatabhisha', 'Purva Bhadrapada',
        'Uttara Bhadrapada', 'Revati'
    ]

    # Nakshatra lords (for Dasha calculation)
    nak_lords = ['Ketu', 'Venus', 'Sun', 'Moon', 'Mars', 'Rahu',
                 'Jupiter', 'Saturn', 'Mercury']
    lord_index = nakshatra_num % 9

    return {
        'name': nakshatras[nakshatra_num],
        'pada': pada_num,
        'lord': nak_lords[lord_index],
        'degree_in_nak': nak_degree
    }

```

## House System Calculations

### Whole Sign Houses (Vedic Default)

In Vedic astrology, the most common house system is **Whole Sign Houses** where the Ascendant sign becomes the entire 1st house, the next sign becomes the 2nd house, etc.
```
def calculate_whole_sign_houses(ascendant_longitude):
    """Calculate 12 house cusps using Whole Sign system"""

    # Ascendant sign number (0-11)
    asc_sign = int(ascendant_longitude / 30)

    houses = {}
    signs = ['Aries', 'Taurus', 'Gemini', 'Cancer', 'Leo', 'Virgo',
             'Libra', 'Scorpio', 'Sagittarius', 'Capricorn', 'Aquarius', 'Pisces']

    for house_num in range(1, 13):
        sign_index = (asc_sign + house_num - 1) % 12
        houses[house_num] = {
            'sign': signs[sign_index],
            'cusp_longitude': sign_index * 30,
            'lord': get_sign_lord(signs[sign_index])
        }

    return houses

def get_sign_lord(sign):
    """Get ruling planet for each sign"""
    lords = {
        'Aries': 'Mars', 'Taurus': 'Venus', 'Gemini': 'Mercury',
        'Cancer': 'Moon', 'Leo': 'Sun', 'Virgo': 'Mercury',
        'Libra': 'Venus', 'Scorpio': 'Mars', 'Sagittarius': 'Jupiter',
        'Capricorn': 'Saturn', 'Aquarius': 'Saturn', 'Pisces': 'Jupiter'
    }
    return lords[sign]

```

### Bhava Madhya (House Centers)

Some astrologers use Bhava Madhya (house centers) for more precise calculations:
```
def calculate_bhava_madhya(ascendant_long, mc_long):
    """Calculate house centers (Bhava Madhya)"""

    bhavas = {}

    # 1st house center = Ascendant
    bhavas[1] = ascendant_long

    # 10th house center = MC (Midheaven)
    bhavas[10] = mc_long

    # Other houses calculated proportionally
    for i in range(2, 10):
        bhavas[i] = (ascendant_long + (i - 1) * 30) % 360

    for i in range(11, 13):
        bhavas[i] = (mc_long + (i - 10) * 30) % 360

    return bhavas

```

## Vimshottari Dasha Calculation

Vimshottari Dasha is the most popular predictive system in Vedic astrology, based on the Moon's nakshatra at birth.

### Dasha Sequence and Durations

| Planet | Years | Days | Signification |
| --- | --- | --- | --- |
| Ketu | 7 | 2,555 | Spirituality, detachment |
| Venus | 20 | 7,300 | Luxury, relationships |
| Sun | 6 | 2,190 | Authority, health |
| Moon | 10 | 3,650 | Mind, emotions |
| Mars | 7 | 2,555 | Energy, property |
| Rahu | 18 | 6,570 | Material desires |
| Jupiter | 16 | 5,840 | Wisdom, expansion |
| Saturn | 19 | 6,935 | Karma, discipline |
| Mercury | 17 | 6,205 | Intelligence, business |

### Implementation Algorithm

```
from datetime import datetime, timedelta

def calculate_vimshottari_dasha(birth_jd, moon_longitude):
    """Calculate complete Vimshottari Dasha sequence"""

    # Dasha periods in years
    dasha_years = {
        'Ketu': 7, 'Venus': 20, 'Sun': 6, 'Moon': 10, 'Mars': 7,
        'Rahu': 18, 'Jupiter': 16, 'Saturn': 19, 'Mercury': 17
    }

    # Dasha sequence
    sequence = ['Ketu', 'Venus', 'Sun', 'Moon', 'Mars',
                'Rahu', 'Jupiter', 'Saturn', 'Mercury']

    # Calculate Moon's nakshatra
    moon_nak = calculate_nakshatra(moon_longitude)

    # Starting planet based on nakshatra lord
    start_planet = moon_nak['lord']
    start_index = sequence.index(start_planet)

    # Calculate balance of first Mahadasha
    nak_completion = moon_nak['degree_in_nak'] / (360/27)  # Fraction completed
    balance_years = dasha_years[start_planet] * (1 - nak_completion)

    # Generate complete 120-year cycle
    dashas = []
    current_date = jd_to_date(birth_jd)

    # First Mahadasha (partial)
    first_end = current_date + timedelta(days=balance_years * 365.25)
    dashas.append({
        'planet': start_planet,
        'start': current_date,
        'end': first_end,
        'years': balance_years,
        'antardashas': calculate_antardashas(start_planet, current_date, first_end, dasha_years)
    })

    # Remaining Mahadashas
    current_date = first_end
    for i in range(1, 9):  # Complete 120-year cycle
        planet = sequence[(start_index + i) % 9]
        years = dasha_years[planet]
        end_date = current_date + timedelta(days=years * 365.25)

        dashas.append({
            'planet': planet,
            'start': current_date,
            'end': end_date,
            'years': years,
            'antardashas': calculate_antardashas(planet, current_date, end_date, dasha_years)
        })

        current_date = end_date

    return dashas

def calculate_antardashas(maha_planet, start_date, end_date, dasha_years):
    """Calculate Antardasha (sub-periods) within Mahadasha"""

    sequence = ['Ketu', 'Venus', 'Sun', 'Moon', 'Mars',
                'Rahu', 'Jupiter', 'Saturn', 'Mercury']

    maha_index = sequence.index(maha_planet)
    total_years = dasha_years[maha_planet]

    antardashas = []
    current_date = start_date

    # Antardasha starts with Mahadasha lord, then follows sequence
    for i in range(9):
        antar_planet = sequence[(maha_index + i) % 9]

        # Proportional duration
        antar_years = (dasha_years[maha_planet] * dasha_years[antar_planet]) / 120
        antar_end = current_date + timedelta(days=antar_years * 365.25)

        antardashas.append({
            'planet': antar_planet,
            'start': current_date,
            'end': antar_end,
            'years': antar_years
        })

        current_date = antar_end

    return antardashas

```

## Yoga Detection Algorithms

Yogas are specific planetary combinations that indicate particular life outcomes. There are 300+ yogas in Vedic astrology.

### Gajakesari Yoga

Formed when Moon and Jupiter are in kendras (1, 4, 7, 10) from each other.
```
def detect_gajakesari_yoga(moon_house, jupiter_house):
    """Detect Gajakesari Yoga"""

    # Calculate house difference
    diff = abs(moon_house - jupiter_house)

    # Kendras are 1, 4, 7, 10 houses apart (or 0 for conjunction)
    kendras = [0, 3, 6, 9]  # 0-indexed: houses 1, 4, 7, 10

    if diff in kendras or (12 - diff) in kendras:
        # Calculate strength
        strength = calculate_yoga_strength(moon_house, jupiter_house)
        return {
            'present': True,
            'name': 'Gajakesari Yoga',
            'description': 'Moon and Jupiter in kendras',
            'strength': strength,
            'effects': 'Wisdom, prosperity, eloquence, fame'
        }
    return {'present': False}

def calculate_yoga_strength(house1, house2):
    """Calculate yoga strength based on house placement"""
    kendra_houses = [1, 4, 7, 10]
    trikona_houses = [1, 5, 9]

    strength = 0
    if house1 in kendra_houses: strength += 2
    if house1 in trikona_houses: strength += 2
    if house2 in kendra_houses: strength += 2
    if house2 in trikona_houses: strength += 2

    if strength >= 6: return 'strong'
    elif strength >= 4: return 'moderate'
    else: return 'weak'

```

### Panch Mahapurusha Yogas

Five great yogas formed when Mars, Mercury, Jupiter, Venus, or Saturn are in kendra in own/exaltation sign.
```
def detect_mahapurusha_yogas(planetary_positions):
    """Detect all five Mahapurusha Yogas"""

    yogas = []

    # Ruchaka Yoga - Mars
    if is_mahapurusha('Mars', planetary_positions):
        yogas.append({
            'name': 'Ruchaka Yoga',
            'planet': 'Mars',
            'effects': 'Courage, valor, commander qualities'
        })

    # Bhadra Yoga - Mercury
    if is_mahapurusha('Mercury', planetary_positions):
        yogas.append({
            'name': 'Bhadra Yoga',
            'planet': 'Mercury',
            'effects': 'Intelligence, scholarly, eloquent'
        })

    # Hamsa Yoga - Jupiter
    if is_mahapurusha('Jupiter', planetary_positions):
        yogas.append({
            'name': 'Hamsa Yoga',
            'planet': 'Jupiter',
            'effects': 'Righteous, wise, spiritual'
        })

    # Malavya Yoga - Venus
    if is_mahapurusha('Venus', planetary_positions):
        yogas.append({
            'name': 'Malavya Yoga',
            'planet': 'Venus',
            'effects': 'Luxurious, artistic, attractive'
        })

    # Shasha Yoga - Saturn
    if is_mahapurusha('Saturn', planetary_positions):
        yogas.append({
            'name': 'Shasha Yoga',
            'planet': 'Saturn',
            'effects': 'Leadership, disciplined, longevity'
        })

    return yogas

def is_mahapurusha(planet, positions):
    """Check if planet forms Mahapurusha Yoga"""

    # Own and exaltation signs for each planet
    conditions = {
        'Mars': {'own': ['Aries', 'Scorpio'], 'exalted': ['Capricorn']},
        'Mercury': {'own': ['Gemini', 'Virgo'], 'exalted': ['Virgo']},
        'Jupiter': {'own': ['Sagittarius', 'Pisces'], 'exalted': ['Cancer']},
        'Venus': {'own': ['Taurus', 'Libra'], 'exalted': ['Pisces']},
        'Saturn': {'own': ['Capricorn', 'Aquarius'], 'exalted': ['Libra']}
    }

    planet_data = positions[planet]
    house = planet_data['house']
    sign = planet_data['sign']

    # Must be in kendra (1, 4, 7, 10)
    kendra = house in [1, 4, 7, 10]

    # Must be in own or exaltation sign
    in_own = sign in conditions[planet]['own']
    in_exaltation = sign in conditions[planet]['exalted']

    return kendra and (in_own or in_exaltation)

```

## Dosha Detection

### Mangal Dosha (Kuja Dosha)

Mangal Dosha occurs when Mars is placed in certain houses (1, 2, 4, 7, 8, 12 from Ascendant or Moon).
```
def detect_mangal_dosha(mars_house_from_asc, mars_house_from_moon):
    """Detect Mangal Dosha (Kuja Dosha)"""

    dosha_houses = [1, 2, 4, 7, 8, 12]

    from_asc = mars_house_from_asc in dosha_houses
    from_moon = mars_house_from_moon in dosha_houses

    if from_asc or from_moon:
        # Calculate severity
        if from_asc and from_moon:
            severity = 'high'
        else:
            severity = 'moderate'

        # Check for cancellations
        cancellations = check_mangal_cancellations(mars_house_from_asc)

        return {
            'present': True,
            'severity': severity,
            'from_ascendant': from_asc,
            'from_moon': from_moon,
            'cancellations': cancellations,
            'effects': 'Delays/challenges in marriage, conflicts'
        }

    return {'present': False}

def check_mangal_cancellations(mars_house):
    """Check conditions that cancel Mangal Dosha"""

    cancellations = []

    # Mars in own sign (Aries/Scorpio)
    # Mars exalted (Capricorn)
    # Both partners have Mangal Dosha
    # Mars in 2nd/12th for certain ascendants

    # These would require full chart context
    # Simplified example:
    if mars_house == 2:
        cancellations.append('Mars in 2nd house (partial cancellation)')

    return cancellations

```

### Kaal Sarp Dosha

Formed when all seven planets are hemmed between Rahu and Ketu.
```
def detect_kaal_sarp_dosha(planetary_positions):
    """Detect Kaal Sarp Dosha"""

    rahu_long = planetary_positions['Rahu']['longitude']
    ketu_long = planetary_positions['Ketu']['longitude']

    # Seven planets (excluding Rahu/Ketu)
    planets = ['Sun', 'Moon', 'Mars', 'Mercury', 'Jupiter', 'Venus', 'Saturn']

    all_between = True
    for planet in planets:
        planet_long = planetary_positions[planet]['longitude']

        # Check if planet is between Rahu and Ketu
        if not is_between_rahu_ketu(planet_long, rahu_long, ketu_long):
            all_between = False
            break

    if all_between:
        dosha_type = determine_kaal_sarp_type(rahu_long)
        return {
            'present': True,
            'type': dosha_type,
            'effects': 'Obstacles, delays, struggles',
            'remedies': ['Rahu-Ketu puja', 'Mantra recitation']
        }

    return {'present': False}

def is_between_rahu_ketu(planet_long, rahu_long, ketu_long):
    """Check if planet is hemmed between Rahu and Ketu"""
    # Handle 360-degree circle wrapping
    if rahu_long < ketu_long:
        return rahu_long <= planet_long <= ketu_long
    else:
        return planet_long >= rahu_long or planet_long <= ketu_long

```

## Planetary Strength Calculations

### Shadbala (Six-Fold Strength)

Shadbala measures planetary strength through six different factors:
1. **Sthana Bala** - Positional strength (exaltation, own sign, etc.)
2. **Dig Bala** - Directional strength (best houses for each planet)
3. **Kala Bala** - Temporal strength (day/night, lunar fortnight)
4. **Chesta Bala** - Motional strength (direct vs retrograde)
5. **Naisargika Bala** - Natural strength (inherent luminosity)
6. **Drik Bala** - Aspectual strength (beneficial vs malefic aspects)

```
def calculate_shadbala(planet, position, chart_data):
    """Calculate Shadbala (six-fold strength) for a planet"""

    total_strength = 0

    # 1. Sthana Bala (Positional)
    total_strength += calculate_sthana_bala(planet, position)

    # 2. Dig Bala (Directional)
    total_strength += calculate_dig_bala(planet, position['house'])

    # 3. Kala Bala (Temporal)
    total_strength += calculate_kala_bala(planet, chart_data)

    # 4. Chesta Bala (Motional)
    total_strength += calculate_chesta_bala(planet, position)

    # 5. Naisargika Bala (Natural)
    total_strength += get_naisargika_bala(planet)

    # 6. Drik Bala (Aspectual)
    total_strength += calculate_drik_bala(planet, chart_data)

    return {
        'total_strength': total_strength,
        'classification': classify_strength(total_strength),
        'components': {
            'sthana': calculate_sthana_bala(planet, position),
            'dig': calculate_dig_bala(planet, position['house']),
            'kala': calculate_kala_bala(planet, chart_data),
            'chesta': calculate_chesta_bala(planet, position),
            'naisargika': get_naisargika_bala(planet),
            'drik': calculate_drik_bala(planet, chart_data)
        }
    }

def calculate_sthana_bala(planet, position):
    """Calculate positional strength"""
    strength = 0

    # Exaltation: Full strength (1000 virupas)
    if position['dignity'] == 'exalted':
        strength = 1000
    # Own sign: 750 virupas
    elif position['dignity'] == 'own':
        strength = 750
    # Friendly: 500 virupas
    elif position['dignity'] == 'friendly':
        strength = 500
    # Neutral: 250 virupas
    elif position['dignity'] == 'neutral':
        strength = 250
    # Debilitated: 0 virupas
    elif position['dignity'] == 'debilitated':
        strength = 0

    return strength

def calculate_dig_bala(planet, house):
    """Calculate directional strength"""

    # Best houses for each planet
    best_houses = {
        'Sun': 10,      # 10th house (MC)
        'Moon': 4,      # 4th house (IC)
        'Mars': 10,     # 10th house
        'Mercury': 1,   # 1st house (Ascendant)
        'Jupiter': 1,   # 1st house
        'Venus': 4,     # 4th house
        'Saturn': 7     # 7th house (Descendant)
    }

    if planet in best_houses:
        best_house = best_houses[planet]

        # Maximum strength when in best house
        if house == best_house:
            return 600
        # Opposite house = minimum strength
        elif house == ((best_house + 6) % 12):
            return 0
        # Proportional for other houses
        else:
            diff = min(abs(house - best_house), 12 - abs(house - best_house))
            return 600 * (1 - diff / 6)

    return 300  # Default for Rahu/Ketu

def get_naisargika_bala(planet):
    """Natural strength (fixed values)"""

    strengths = {
        'Sun': 600,
        'Moon': 510,
        'Mars': 420,
        'Mercury': 390,
        'Jupiter': 510,
        'Venus': 420,
        'Saturn': 300
    }

    return strengths.get(planet, 0)

```

### Ashtakavarga

Ashtakavarga is a predictive system that assigns points (bindus) to each house based on planetary relationships.
```
def calculate_ashtakavarga(planetary_positions):
    """Calculate Ashtakavarga points for all houses"""

    # Each planet contributes points based on position from itself and others
    # This is a simplified version - full calculation is complex

    house_points = {i: 0 for i in range(1, 13)}

    planets = ['Sun', 'Moon', 'Mars', 'Mercury', 'Jupiter', 'Venus', 'Saturn']

    for planet in planets:
        planet_house = planetary_positions[planet]['house']

        # Add points based on benefic/malefic nature and aspects
        benefic_houses = get_benefic_houses_from(planet, planet_house)

        for house in benefic_houses:
            house_points[house] += 1

    # Classify houses based on total points
    for house in house_points:
        points = house_points[house]
        house_points[house] = {
            'points': points,
            'strength': 'strong' if points >= 28 else 'weak' if points < 25 else 'moderate'
        }

    return house_points

```

## Production Implementation Tips

### 1. Ephemeris Data Management

- Use XALEN Ephemeris for accuracy
- Implement fallback to NASA JPL Horizons
- Cache ephemeris lookups (planetary positions don't change)
- Handle edge cases: pre-1800 dates, post-2100 dates

### 2. Performance Optimization

- Pre-calculate and cache birth charts
- Use vectorized operations for bulk calculations
- Implement lazy loading for divisional charts
- Optimize yoga detection with early exits

### 3. Accuracy Considerations

- Always use proper timezone databases
- Handle historical timezone changes
- Account for Daylight Saving Time
- Validate location coordinates
- Use exact birth time when available

### 4. Testing Strategy

- Cross-verify with established software (Jagannatha Hora, Parashara's Light)
- Test edge cases: midnight births, date line crossings
- Validate yoga detection against known charts
- Implement regression tests for calculation changes

#### Using Vedika API

All these complex calculations are available via [Vedika API](https://vedika.io/) with just a simple REST call. No need to implement XALEN Ephemeris, Ayanamsa corrections, or yoga algorithms yourself.
```
import requests

response = requests.post('https://api.vedika.io/v2/astrology/birth-chart',
    headers={'Authorization': 'Bearer YOUR_KEY'},
    json={
        'date': '1990-06-15',
        'time': '14:30',
        'latitude': 28.6139,
        'longitude': 77.2090,
        'timezone': 5.5
    }
)

chart = response.json()
# Get all calculations instantly!
```

## Skip the Complex Math - Use Vedika API

Get all these calculations instantly via REST API. FREE Sandbox + plans from $12/month.

[Get API Key](https://vedika.io/dashboard) [Read Docs](https://vedika.io/docs)

---
_Full page (always current): [https://vedika.io/guides/vedic-astrology-calculations-guide](https://vedika.io/guides/vedic-astrology-calculations-guide) · [Vedika docs](https://vedika.io/docs)_
