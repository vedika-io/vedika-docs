---
title: "Python CLI Tool | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/python-cli
canonical: https://vedika.io/docs/sdk/python-cli
---

# CLI Tool

Command line interface for Vedika API

## Installation

```
# Install with CLI support
pip install vedika-sdk[cli]

# Set API key
export VEDIKA_API_KEY=vk_live_your_key

# Verify installation
vedika --version
```

## Birth Chart

```
# Basic chart
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090

# With timezone
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --tz "Asia/Kolkata"

# Output as JSON
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --json

# Save to file
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 -o chart.json
```

Birth Chart Analysis

==================

Sun Sign: Taurus

Moon Sign: Aries

Ascendant: Cancer

Nakshatra: Ashwini

Planetary Positions:

Sun Taurus 24.38deg

Moon Aries 12.15deg

Mars Aquarius 8.42deg

...

## Panchang

```
# Today's panchang (uses current location if available)
vedika panchang

# Specific date and location
vedika panchang --date "2026-01-07" --lat 28.6139 --lng 77.2090

# Show Rahu Kaal
vedika panchang --date "2026-01-07" --lat 28.6139 --lng 77.2090 --rahu-kaal

# Full details
vedika panchang --date "2026-01-07" --lat 28.6139 --lng 77.2090 --full
```

Panchang for 2026-01-07

======================

Tithi: Shukla Saptami

Nakshatra: Pushya

Yoga: Siddha

Karana: Bava

Sunrise: 07:14 AM

Sunset: 05:38 PM

Rahu Kaal: 12:26 PM - 01:48 PM

## Kundli Matching

```
# Match two charts
vedika match \
  --bride-date "1995-03-20" --bride-time "14:30" --bride-lat 28.6139 --bride-lng 77.2090 \
  --groom-date "1992-07-15" --groom-time "06:00" --groom-lat 19.0760 --groom-lng 72.8777

# With detailed guna breakdown
vedika match \
  --bride-date "1995-03-20" --bride-time "14:30" --bride-lat 28.6139 --bride-lng 77.2090 \
  --groom-date "1992-07-15" --groom-time "06:00" --groom-lat 19.0760 --groom-lng 72.8777 \
  --detailed
```

## AI Query

```
# Ask a question
vedika ask "What are my career prospects?" \
  --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090

# Interactive chat mode
vedika chat --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090

# Stream response
vedika ask "Analyze my chart" \
  --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 \
  --stream
```

## Dasha Periods

```
# Get current dasha
vedika dasha --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090

# Full Mahadasha timeline
vedika dasha --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --full

# Dasha for specific year
vedika dasha --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --year 2030
```

## Configuration

```
# Save default location
vedika config set location --lat 28.6139 --lng 77.2090 --tz "Asia/Kolkata"

# Now panchang uses default location
vedika panchang

# View config
vedika config show

# Set API key
vedika config set api-key vk_live_your_key

# Clear config
vedika config clear
```

## Batch Processing

```
# Process from CSV file
vedika batch charts --input birth_data.csv --output charts.json

# CSV format:
# datetime,latitude,longitude
# 1990-05-15T10:30:00+05:30,28.6139,77.2090
# 1985-03-20T14:00:00+05:30,19.0760,72.8777

# Parallel processing
vedika batch charts --input birth_data.csv --output charts.json --workers 10
```

## Output Formats

```
# Default: Pretty table
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090

# JSON output
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --json

# CSV output
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --csv

# Quiet mode (minimal output)
vedika chart --date "1990-05-15" --time "10:30" --lat 28.6139 --lng 77.2090 --quiet
```

## All Commands

```
vedika --help

Commands:
  chart      Generate birth chart
  panchang   Get daily panchang
  match      Kundli matching
  dasha      Dasha periods
  dosha      Check doshas (Mangal, Kaal Sarp)
  ask        AI-powered question
  chat       Interactive chat mode
  batch      Batch processing
  config     Configure CLI settings
  usage      Check API usage

Global Options:
  --api-key    Override API key
  --json       Output as JSON
  --quiet      Minimal output
  --verbose    Detailed output
  --version    Show version
```

## Related Topics

[ **

#### Python Quickstart

SDK usage in code

](https://vedika.io/docs/sdk/python-quickstart) [ **

#### Async Usage

Batch processing

](https://vedika.io/docs/sdk/python-async)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-cli](https://vedika.io/docs/sdk/python-cli) · [Vedika docs](https://vedika.io/docs)_
