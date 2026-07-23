---
title: "Mobile App Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/mobile-app
canonical: https://vedika.io/docs/sdk/mobile-app
---

# Mobile App Integration

React Native and Flutter guides

## Architecture

**Important:** Never include API keys in mobile app code. Always proxy through your backend server.

Mobile App

Your Backend

Vedika API

## **React Native

### API Service

```
// services/vedika.ts
const API_BASE = 'https://your-backend.com/api';

export interface BirthDetails {
  datetime: string;
  latitude: number;
  longitude: number;
}

export interface BirthChart {
  sunSign: string;
  moonSign: string;
  ascendant: string;
  planets: Planet[];
}

export async function getBirthChart(birthDetails: BirthDetails): Promise<BirthChart> {
  const response = await fetch(`${API_BASE}/chart`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(birthDetails)
  });

  if (!response.ok) {
    throw new Error('Failed to fetch chart');
  }

  return response.json();
}

export async function getPanchang(date: string, lat: number, lng: number) {
  const response = await fetch(
    `${API_BASE}/panchang?date=${date}&lat=${lat}&lng=${lng}`
  );
  return response.json();
}
```

### Hook

```
// hooks/useVedika.ts
import { useState, useCallback } from 'react';
import * as vedikaService from '../services/vedika';

export function useVedika() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchChart = useCallback(async (birthDetails: vedikaService.BirthDetails) => {
    setLoading(true);
    setError(null);

    try {
      return await vedikaService.getBirthChart(birthDetails);
    } catch (e) {
      setError(e instanceof Error ? e.message : 'Unknown error');
      return null;
    } finally {
      setLoading(false);
    }
  }, []);

  return { fetchChart, loading, error };
}
```

### Screen Component

```
// screens/ChartScreen.tsx
import React, { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet } from 'react-native';
import DateTimePicker from '@react-native-community/datetimepicker';
import { useVedika } from '../hooks/useVedika';

export function ChartScreen() {
  const [date, setDate] = useState(new Date());
  const [chart, setChart] = useState(null);
  const { fetchChart, loading, error } = useVedika();

  const handleGenerate = async () => {
    const result = await fetchChart({
      datetime: date.toISOString(),
      latitude: 28.6139,
      longitude: 77.2090
    });
    if (result) setChart(result);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Birth Chart</Text>

      <DateTimePicker
        value={date}
        mode="datetime"
        onChange={(e, d) => d && setDate(d)}
      />

      <Button
        title={loading ? 'Generating...' : 'Generate Chart'}
        onPress={handleGenerate}
        disabled={loading}
      />

      {error && <Text style={styles.error}>{error}</Text>}

      {chart && (
        <View style={styles.chart}>
          <Text>Sun Sign: {chart.sunSign}</Text>
          <Text>Moon Sign: {chart.moonSign}</Text>
          <Text>Ascendant: {chart.ascendant}</Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  title: { fontSize: 24, fontWeight: 'bold', marginBottom: 20 },
  chart: { marginTop: 20, padding: 15, backgroundColor: '#f0f0f0', borderRadius: 10 },
  error: { color: 'red', marginTop: 10 }
});
```

## **Flutter

### API Service

```
// lib/services/vedika_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class VedikaService {
  static const String baseUrl = 'https://your-backend.com/api';

  Future<BirthChart> getBirthChart(BirthDetails details) async {
    final response = await http.post(
      Uri.parse('$baseUrl/chart'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(details.toJson()),
    );

    if (response.statusCode == 200) {
      return BirthChart.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to fetch chart');
    }
  }

  Future<Panchang> getPanchang(String date, double lat, double lng) async {
    final response = await http.get(
      Uri.parse('$baseUrl/panchang?date=$date&lat=$lat&lng=$lng'),
    );

    if (response.statusCode == 200) {
      return Panchang.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to fetch panchang');
    }
  }
}

class BirthDetails {
  final String datetime;
  final double latitude;
  final double longitude;

  BirthDetails({required this.datetime, required this.latitude, required this.longitude});

  Map<String, dynamic> toJson() => {
    'datetime': datetime,
    'latitude': latitude,
    'longitude': longitude,
  };
}

class BirthChart {
  final String sunSign;
  final String moonSign;
  final String ascendant;

  BirthChart({required this.sunSign, required this.moonSign, required this.ascendant});

  factory BirthChart.fromJson(Map<String, dynamic> json) => BirthChart(
    sunSign: json['sunSign'],
    moonSign: json['moonSign'],
    ascendant: json['ascendant'],
  );
}
```

### Screen Widget

```
// lib/screens/chart_screen.dart
import 'package:flutter/material.dart';
import '../services/vedika_service.dart';

class ChartScreen extends StatefulWidget {
  @override
  _ChartScreenState createState() => _ChartScreenState();
}

class _ChartScreenState extends State<ChartScreen> {
  final VedikaService _vedika = VedikaService();
  BirthChart? _chart;
  bool _loading = false;
  DateTime _birthDate = DateTime.now();

  Future<void> _generateChart() async {
    setState(() => _loading = true);

    try {
      final chart = await _vedika.getBirthChart(BirthDetails(
        datetime: _birthDate.toIso8601String(),
        latitude: 28.6139,
        longitude: 77.2090,
      ));
      setState(() => _chart = chart);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Error: $e')),
      );
    } finally {
      setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Birth Chart')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            ElevatedButton(
              onPressed: () async {
                final date = await showDatePicker(
                  context: context,
                  initialDate: _birthDate,
                  firstDate: DateTime(1900),
                  lastDate: DateTime.now(),
                );
                if (date != null) setState(() => _birthDate = date);
              },
              child: Text('Select Birth Date'),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: _loading ? null : _generateChart,
              child: Text(_loading ? 'Generating...' : 'Generate Chart'),
            ),
            SizedBox(height: 20),
            if (_chart != null) ...[
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    children: [
                      Text('Sun Sign: ${_chart!.sunSign}'),
                      Text('Moon Sign: ${_chart!.moonSign}'),
                      Text('Ascendant: ${_chart!.ascendant}'),
                    ],
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

## Push Notifications

```
// Backend: Send daily horoscope notifications
async function sendDailyHoroscope(userToken, zodiacSign) {
  const horoscope = await vedika.dailyHoroscope({ sign: zodiacSign });

  await admin.messaging().send({
    token: userToken,
    notification: {
      title: `${zodiacSign} Daily Horoscope`,
      body: horoscope.prediction.substring(0, 100) + '...'
    },
    data: {
      type: 'horoscope',
      sign: zodiacSign,
      date: new Date().toISOString()
    }
  });
}
```

## Next Steps

[ **

#### Horoscope App

Complete app tutorial

](https://vedika.io/docs/sdk/horoscope-app) [ **

#### Streaming API

Real-time responses

](https://vedika.io/docs/sdk/streaming)

---
_Full page (always current): [https://vedika.io/docs/sdk/mobile-app](https://vedika.io/docs/sdk/mobile-app) · [Vedika docs](https://vedika.io/docs)_
