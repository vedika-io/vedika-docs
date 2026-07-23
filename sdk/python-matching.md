---
title: "Python Kundli Matching | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/python-matching
canonical: https://vedika.io/docs/sdk/python-matching
---

# Kundli Matching

Marriage compatibility in Python

## Basic Usage

```
from vedika import VedikaClient

client = VedikaClient()

match = client.kundli_match({
    "bride": {
        "datetime": "1995-03-20T14:30:00+05:30",
        "latitude": 28.6139,
        "longitude": 77.2090
    },
    "groom": {
        "datetime": "1992-07-15T06:00:00+05:30",
        "latitude": 19.0760,
        "longitude": 72.8777
    }
})

print(f"Total Score: {match.total_score}/{match.max_score}")
print(f"Percentage: {match.percentage}%")
print(f"Verdict: {match.verdict}")
```

## Guna Breakdown

```
# Access individual gunas
for guna in match.gunas:
    status = "Matched" if guna.is_matched else "Not Matched"
    print(f"{guna.name}: {guna.obtained_points}/{guna.max_points} - {status}")

# Output:
# Varna: 1/1 - Matched
# Vashya: 2/2 - Matched
# Tara: 3/3 - Matched
# Yoni: 3/4 - Not Matched
# Graha Maitri: 5/5 - Matched
# Gana: 6/6 - Matched
# Bhakoot: 7/7 - Matched
# Nadi: 8/8 - Matched
```

## Manglik Compatibility

```
# Check Manglik status
print(f"Bride Manglik: {match.manglik_status.bride_is_manglik}")
print(f"Groom Manglik: {match.manglik_status.groom_is_manglik}")
print(f"Compatible: {match.manglik_status.is_compatible}")

# Standalone Manglik check
mangal = client.mangal_dosha({
    "datetime": "1995-03-20T14:30:00+05:30",
    "latitude": 28.6139,
    "longitude": 77.2090
})

print(f"Is Manglik: {mangal.is_manglik}")
print(f"Severity: {mangal.severity}")
print(f"Cancellations: {mangal.cancellations}")
```

## South Indian (Porutham)

```
# Use Dasa Porutham system
match = client.kundli_match({
    "bride": bride_details,
    "groom": groom_details,
    "system": "porutham"  # South Indian method
})

# Access 10 poruthams
for porutham in match.poruthams:
    status = "Yes" if porutham.matched else "No"
    print(f"{porutham.name}: {status}")

# Output:
# Dina: Yes
# Gana: Yes
# Mahendra: No
# Stree Deergha: Yes
# Yoni: Yes
# Rasi: Yes
# Rasiadhipati: Yes
# Vasya: Yes
# Rajju: Yes
# Vedha: Yes
```

## Recommendations & Remedies

```
match = client.kundli_match({
    "bride": bride_details,
    "groom": groom_details,
    "include_remedies": True
})

# Access recommendations
print("Strengths:")
for strength in match.recommendations.strengths:
    print(f"  - {strength}")

print("\nConcerns:")
for concern in match.recommendations.concerns:
    print(f"  - {concern}")

print("\nRemedies:")
for remedy in match.recommendations.remedies:
    print(f"  - {remedy}")
```

## Batch Matching

```
from vedika import AsyncVedikaClient
import asyncio

async def match_multiple_profiles(bride, grooms):
    """Match one bride with multiple groom profiles."""
    async with AsyncVedikaClient() as client:
        tasks = [
            client.kundli_match({"bride": bride, "groom": groom})
            for groom in grooms
        ]
        results = await asyncio.gather(*tasks)

    # Sort by score
    ranked = sorted(
        zip(grooms, results),
        key=lambda x: x[1].total_score,
        reverse=True
    )

    return ranked

# Usage
bride = {"datetime": "1995-03-20T14:30:00+05:30", "latitude": 28.6139, "longitude": 77.2090}
grooms = [
    {"datetime": "1992-07-15T06:00:00+05:30", "latitude": 19.0760, "longitude": 72.8777},
    {"datetime": "1990-12-25T10:00:00+05:30", "latitude": 13.0827, "longitude": 80.2707},
    {"datetime": "1993-08-08T15:30:00+05:30", "latitude": 22.5726, "longitude": 88.3639},
]

ranked_matches = asyncio.run(match_multiple_profiles(bride, grooms))
for groom, match in ranked_matches:
    print(f"Score: {match.total_score} - {match.verdict}")
```

## Generate Matching Report

```
def generate_match_report(match):
    """Generate a detailed matching report."""
    report = []
    report.append("=" * 50)
    report.append("KUNDLI MATCHING REPORT")
    report.append("=" * 50)

    report.append(f"\nOverall Score: {match.total_score}/{match.max_score} ({match.percentage}%)")
    report.append(f"Verdict: {match.verdict}\n")

    report.append("GUNA ANALYSIS:")
    report.append("-" * 30)
    for guna in match.gunas:
        status = "MATCH" if guna.is_matched else "----"
        report.append(f"{guna.name:15} {guna.obtained_points}/{guna.max_points}  [{status}]")

    report.append(f"\nMANGLIK STATUS:")
    report.append("-" * 30)
    report.append(f"Bride:  {'Manglik' if match.manglik_status.bride_is_manglik else 'Non-Manglik'}")
    report.append(f"Groom:  {'Manglik' if match.manglik_status.groom_is_manglik else 'Non-Manglik'}")

    if match.recommendations.remedies:
        report.append(f"\nRECOMMENDED REMEDIES:")
        report.append("-" * 30)
        for remedy in match.recommendations.remedies:
            report.append(f"  * {remedy}")

    return "\n".join(report)

# Usage
match = client.kundli_match({"bride": bride, "groom": groom, "include_remedies": True})
print(generate_match_report(match))
```

## Related APIs

[ **

#### Birth Charts

Individual horoscopes

](https://vedika.io/docs/sdk/python-charts) [ **

#### Matching Portal

Build matrimonial sites

](https://vedika.io/docs/sdk/matching-portal)

---
_Full page (always current): [https://vedika.io/docs/sdk/python-matching](https://vedika.io/docs/sdk/python-matching) · [Vedika docs](https://vedika.io/docs)_
