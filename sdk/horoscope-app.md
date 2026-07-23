---
title: "Build a Horoscope App | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/horoscope-app
canonical: https://vedika.io/docs/sdk/horoscope-app
---

# Build a Horoscope App

Daily, weekly, and monthly predictions

## App Features

#### Daily Horoscope

Fresh predictions every day

#### Weekly Forecast

Plan your week ahead

#### Monthly Overview

Long-term trends

#### Push Notifications

Morning reminders

#### Personalized

Based on birth chart

#### Social Sharing

Share with friends

## Backend API (Node.js)

```
// server/routes/horoscope.ts
import express from 'express';
import { VedikaClient } from '@vedika-io/sdk';
import { cache } from '../services/cache';

const router = express.Router();
const vedika = new VedikaClient();

const ZODIAC_SIGNS = [
  'aries', 'taurus', 'gemini', 'cancer',
  'leo', 'virgo', 'libra', 'scorpio',
  'sagittarius', 'capricorn', 'aquarius', 'pisces'
];

// Daily horoscope for all signs (cached)
router.get('/daily', async (req, res) => {
  const today = new Date().toISOString().split('T')[0];
  const cacheKey = `horoscope:daily:${today}`;

  // Try cache first
  const cached = await cache.get(cacheKey);
  if (cached) {
    return res.json(JSON.parse(cached));
  }

  // Fetch all signs in parallel
  const predictions = await Promise.all(
    ZODIAC_SIGNS.map(async sign => {
      const horoscope = await vedika.dailyHoroscope({ sign, date: today });
      return {
        sign,
        prediction: horoscope.prediction,
        luckyNumber: horoscope.luckyNumber,
        luckyColor: horoscope.luckyColor,
        mood: horoscope.mood,
        compatibility: horoscope.compatibility
      };
    })
  );

  const result = { date: today, predictions };

  // Cache until midnight
  const secondsUntilMidnight = getSecondsUntilMidnight();
  await cache.set(cacheKey, JSON.stringify(result), secondsUntilMidnight);

  res.json(result);
});

// Single sign daily horoscope
router.get('/daily/:sign', async (req, res) => {
  const { sign } = req.params;
  const today = new Date().toISOString().split('T')[0];

  if (!ZODIAC_SIGNS.includes(sign.toLowerCase())) {
    return res.status(400).json({ error: 'Invalid zodiac sign' });
  }

  const cacheKey = `horoscope:daily:${today}:${sign}`;
  const cached = await cache.get(cacheKey);

  if (cached) {
    return res.json(JSON.parse(cached));
  }

  const horoscope = await vedika.dailyHoroscope({
    sign: sign.toLowerCase(),
    date: today
  });

  await cache.set(cacheKey, JSON.stringify(horoscope), getSecondsUntilMidnight());
  res.json(horoscope);
});

// Weekly horoscope
router.get('/weekly/:sign', async (req, res) => {
  const { sign } = req.params;
  const weekStart = getWeekStart();

  const horoscope = await vedika.weeklyHoroscope({
    sign: sign.toLowerCase(),
    weekStart
  });

  res.json(horoscope);
});

// Monthly horoscope
router.get('/monthly/:sign', async (req, res) => {
  const { sign } = req.params;
  const month = new Date().toISOString().slice(0, 7); // YYYY-MM

  const horoscope = await vedika.monthlyHoroscope({
    sign: sign.toLowerCase(),
    month
  });

  res.json(horoscope);
});

// Personalized horoscope based on birth chart
router.get('/personalized', async (req, res) => {
  const user = req.user; // Authenticated user

  if (!user.birthDetails) {
    return res.status(400).json({
      error: 'Birth details required for personalized horoscope'
    });
  }

  const today = new Date().toISOString().split('T')[0];

  const [chart, panchang, transits] = await Promise.all([
    vedika.birthChart(user.birthDetails),
    vedika.panchang({
      date: today,
      latitude: user.birthDetails.latitude,
      longitude: user.birthDetails.longitude
    }),
    vedika.currentTransits(user.birthDetails)
  ]);

  // Generate personalized prediction using AI
  const prediction = await vedika.query({
    question: 'What should I focus on today based on current planetary transits?',
    birthDetails: user.birthDetails
  });

  res.json({
    date: today,
    sunSign: chart.sunSign,
    moonSign: chart.moonSign,
    ascendant: chart.ascendant,
    todayPanchang: {
      tithi: panchang.tithi.name,
      nakshatra: panchang.nakshatra.name,
      yoga: panchang.yoga.name
    },
    activeTransits: transits.filter(t => t.isActive),
    prediction: prediction.answer,
    advice: prediction.advice
  });
});

function getSecondsUntilMidnight() {
  const now = new Date();
  const midnight = new Date(now);
  midnight.setHours(24, 0, 0, 0);
  return Math.floor((midnight - now) / 1000);
}

function getWeekStart() {
  const now = new Date();
  const day = now.getDay();
  const diff = now.getDate() - day + (day === 0 ? -6 : 1);
  return new Date(now.setDate(diff)).toISOString().split('T')[0];
}

export default router;
```

## React Native App

### Home Screen

```
// screens/HomeScreen.tsx
import React, { useState, useEffect } from 'react';
import {
  View, Text, ScrollView, TouchableOpacity,
  StyleSheet, RefreshControl, Image
} from 'react-native';
import { useAuth } from '../hooks/useAuth';
import { ZodiacCard } from '../components/ZodiacCard';

const ZODIAC_INFO = {
  aries: { icon: require('../assets/aries.png'), color: '#FF6B6B', dates: 'Mar 21 - Apr 19' },
  taurus: { icon: require('../assets/taurus.png'), color: '#4ECDC4', dates: 'Apr 20 - May 20' },
  gemini: { icon: require('../assets/gemini.png'), color: '#FFE66D', dates: 'May 21 - Jun 20' },
  cancer: { icon: require('../assets/cancer.png'), color: '#95E1D3', dates: 'Jun 21 - Jul 22' },
  leo: { icon: require('../assets/leo.png'), color: '#F38181', dates: 'Jul 23 - Aug 22' },
  virgo: { icon: require('../assets/virgo.png'), color: '#AA96DA', dates: 'Aug 23 - Sep 22' },
  libra: { icon: require('../assets/libra.png'), color: '#FCBAD3', dates: 'Sep 23 - Oct 22' },
  scorpio: { icon: require('../assets/scorpio.png'), color: '#A8D8EA', dates: 'Oct 23 - Nov 21' },
  sagittarius: { icon: require('../assets/sagittarius.png'), color: '#F9ED69', dates: 'Nov 22 - Dec 21' },
  capricorn: { icon: require('../assets/capricorn.png'), color: '#B5EAD7', dates: 'Dec 22 - Jan 19' },
  aquarius: { icon: require('../assets/aquarius.png'), color: '#C7CEEA', dates: 'Jan 20 - Feb 18' },
  pisces: { icon: require('../assets/pisces.png'), color: '#E2F0CB', dates: 'Feb 19 - Mar 20' }
};

export function HomeScreen({ navigation }) {
  const { user } = useAuth();
  const [horoscopes, setHoroscopes] = useState([]);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    loadHoroscopes();
  }, []);

  async function loadHoroscopes() {
    const response = await fetch(`${API_URL}/horoscope/daily`);
    const data = await response.json();
    setHoroscopes(data.predictions);
  }

  async function onRefresh() {
    setRefreshing(true);
    await loadHoroscopes();
    setRefreshing(false);
  }

  // Show user's sign first if logged in
  const sortedSigns = user?.zodiacSign
    ? [user.zodiacSign, ...Object.keys(ZODIAC_INFO).filter(s => s !== user.zodiacSign)]
    : Object.keys(ZODIAC_INFO);

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      <View style={styles.header}>
        <Text style={styles.date}>{formatDate(new Date())}</Text>
        <Text style={styles.title}>Daily Horoscope</Text>
      </View>

      {user?.zodiacSign && (
        <TouchableOpacity
          style={[styles.featured, { backgroundColor: ZODIAC_INFO[user.zodiacSign].color }]}
          onPress={() => navigation.navigate('Detail', { sign: user.zodiacSign })}
        >
          <Image source={ZODIAC_INFO[user.zodiacSign].icon} style={styles.featuredIcon} />
          <View style={styles.featuredContent}>
            <Text style={styles.featuredSign}>{capitalize(user.zodiacSign)}</Text>
            <Text style={styles.featuredPreview}>
              {horoscopes.find(h => h.sign === user.zodiacSign)?.prediction.slice(0, 100)}...
            </Text>
          </View>
        </TouchableOpacity>
      )}

      <View style={styles.grid}>
        {sortedSigns.map(sign => (
          <ZodiacCard
            key={sign}
            sign={sign}
            info={ZODIAC_INFO[sign]}
            horoscope={horoscopes.find(h => h.sign === sign)}
            onPress={() => navigation.navigate('Detail', { sign })}
            isUserSign={sign === user?.zodiacSign}
          />
        ))}
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#F8F9FA' },
  header: { padding: 20, paddingTop: 60 },
  date: { fontSize: 14, color: '#666', marginBottom: 4 },
  title: { fontSize: 28, fontWeight: 'bold', color: '#1A1A2E' },
  featured: { margin: 20, borderRadius: 16, padding: 20, flexDirection: 'row' },
  featuredIcon: { width: 60, height: 60 },
  featuredContent: { flex: 1, marginLeft: 16 },
  featuredSign: { fontSize: 24, fontWeight: 'bold', color: '#FFF' },
  featuredPreview: { fontSize: 14, color: 'rgba(255,255,255,0.9)', marginTop: 8 },
  grid: { flexDirection: 'row', flexWrap: 'wrap', padding: 10 }
});
```

### Detail Screen

```
// screens/DetailScreen.tsx
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet, Share } from 'react-native';
import { TabView, SceneMap, TabBar } from 'react-native-tab-view';

export function DetailScreen({ route }) {
  const { sign } = route.params;
  const [index, setIndex] = useState(0);
  const [routes] = useState([
    { key: 'daily', title: 'Today' },
    { key: 'weekly', title: 'This Week' },
    { key: 'monthly', title: 'This Month' }
  ]);

  const [data, setData] = useState({
    daily: null,
    weekly: null,
    monthly: null
  });

  useEffect(() => {
    loadHoroscope('daily');
  }, []);

  useEffect(() => {
    const period = routes[index].key;
    if (!data[period]) {
      loadHoroscope(period);
    }
  }, [index]);

  async function loadHoroscope(period) {
    const response = await fetch(`${API_URL}/horoscope/${period}/${sign}`);
    const horoscope = await response.json();
    setData(prev => ({ ...prev, [period]: horoscope }));
  }

  async function shareHoroscope() {
    const period = routes[index].key;
    const horoscope = data[period];

    await Share.share({
      message: `${capitalize(sign)} ${period} horoscope: ${horoscope.prediction}\n\nGet your horoscope at MyAstrologyApp`,
      title: `${capitalize(sign)} Horoscope`
    });
  }

  const DailyScene = () => (
    <HoroscopeContent
      horoscope={data.daily}
      showExtras={true}
    />
  );

  const WeeklyScene = () => (
    <HoroscopeContent horoscope={data.weekly} />
  );

  const MonthlyScene = () => (
    <HoroscopeContent horoscope={data.monthly} />
  );

  return (
    <View style={styles.container}>
      <TabView
        navigationState={{ index, routes }}
        renderScene={SceneMap({
          daily: DailyScene,
          weekly: WeeklyScene,
          monthly: MonthlyScene
        })}
        onIndexChange={setIndex}
        renderTabBar={props => (
          <TabBar
            {...props}
            style={styles.tabBar}
            indicatorStyle={styles.indicator}
            labelStyle={styles.tabLabel}
          />
        )}
      />
    </View>
  );
}

function HoroscopeContent({ horoscope, showExtras = false }) {
  if (!horoscope) {
    return <ActivityIndicator size="large" />;
  }

  return (
    <ScrollView style={styles.content}>
      <Text style={styles.prediction}>{horoscope.prediction}</Text>

      {showExtras && (
        <View style={styles.extras}>
          <View style={styles.extraItem}>
            <Text style={styles.extraLabel}>Lucky Number</Text>
            <Text style={styles.extraValue}>{horoscope.luckyNumber}</Text>
          </View>
          <View style={styles.extraItem}>
            <Text style={styles.extraLabel}>Lucky Color</Text>
            <View style={[styles.colorDot, { backgroundColor: horoscope.luckyColor }]} />
          </View>
          <View style={styles.extraItem}>
            <Text style={styles.extraLabel}>Mood</Text>
            <Text style={styles.extraValue}>{horoscope.mood}</Text>
          </View>
          <View style={styles.extraItem}>
            <Text style={styles.extraLabel}>Best Match</Text>
            <Text style={styles.extraValue}>{horoscope.compatibility}</Text>
          </View>
        </View>
      )}
    </ScrollView>
  );
}
```

## Push Notifications

```
// server/services/notifications.ts
import * as admin from 'firebase-admin';
import { VedikaClient } from '@vedika-io/sdk';
import cron from 'node-cron';

const vedika = new VedikaClient();

// Send morning horoscope at 7 AM user's local time
cron.schedule('0 * * * *', async () => {
  // Get users where it's 7 AM in their timezone
  const users = await db.query(`
    SELECT id, fcm_token, zodiac_sign, timezone
    FROM users
    WHERE fcm_token IS NOT NULL
      AND EXTRACT(HOUR FROM NOW() AT TIME ZONE timezone) = 7
  `);

  for (const user of users.rows) {
    try {
      const horoscope = await vedika.dailyHoroscope({
        sign: user.zodiac_sign
      });

      await admin.messaging().send({
        token: user.fcm_token,
        notification: {
          title: `Good morning! Your ${capitalize(user.zodiac_sign)} forecast`,
          body: horoscope.prediction.slice(0, 100) + '...'
        },
        data: {
          type: 'daily_horoscope',
          sign: user.zodiac_sign
        },
        android: {
          notification: {
            icon: 'ic_star',
            color: '#FFD700'
          }
        },
        apns: {
          payload: {
            aps: {
              badge: 1,
              sound: 'default'
            }
          }
        }
      });

    } catch (error) {
      console.error(`Failed to send notification to ${user.id}:`, error);
    }
  }
});

// Client-side notification handling
// React Native
import messaging from '@react-native-firebase/messaging';

async function requestPermission() {
  const authStatus = await messaging().requestPermission();
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;

  if (enabled) {
    const token = await messaging().getToken();
    await fetch(`${API_URL}/user/fcm-token`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token })
    });
  }
}

messaging().onMessage(async remoteMessage => {
  // Handle foreground notification
  Alert.alert(
    remoteMessage.notification.title,
    remoteMessage.notification.body
  );
});

messaging().setBackgroundMessageHandler(async remoteMessage => {
  // Handle background notification
  console.log('Background message:', remoteMessage);
});
```

## iOS Home Screen Widget

```
// HoroscopeWidget.swift
import WidgetKit
import SwiftUI

struct HoroscopeEntry: TimelineEntry {
    let date: Date
    let sign: String
    let prediction: String
    let luckyNumber: Int
}

struct Provider: TimelineProvider {
    func getTimeline(in context: Context, completion: @escaping (Timeline<HoroscopeEntry>) -> Void) {
        Task {
            let sign = UserDefaults.standard.string(forKey: "userZodiacSign") ?? "aries"
            let horoscope = try await fetchHoroscope(sign: sign)

            let entry = HoroscopeEntry(
                date: Date(),
                sign: sign,
                prediction: horoscope.prediction,
                luckyNumber: horoscope.luckyNumber
            )

            // Refresh at midnight
            let midnight = Calendar.current.startOfDay(for: Date().addingTimeInterval(86400))
            let timeline = Timeline(entries: [entry], policy: .after(midnight))
            completion(timeline)
        }
    }
}

struct HoroscopeWidgetView: View {
    let entry: HoroscopeEntry

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Image(entry.sign)
                    .resizable()
                    .frame(width: 24, height: 24)
                Text(entry.sign.capitalized)
                    .font(.headline)
            }

            Text(entry.prediction)
                .font(.caption)
                .lineLimit(3)

            Spacer()

            HStack {
                Text("Lucky #")
                    .font(.caption2)
                    .foregroundColor(.secondary)
                Text("\(entry.luckyNumber)")
                    .font(.caption)
                    .bold()
            }
        }
        .padding()
    }
}

@main
struct HoroscopeWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "HoroscopeWidget", provider: Provider()) { entry in
            HoroscopeWidgetView(entry: entry)
        }
        .configurationDisplayName("Daily Horoscope")
        .description("Your daily zodiac prediction")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

## Monetization Strategies

#### **Ad-Supported Free Tier

- - Daily horoscope with ads
- - Banner ads between sections
- - Rewarded video for weekly forecast

#### **Premium Subscription

- - Ad-free experience
- - Personalized predictions
- - Weekly & monthly forecasts
- - Push notifications

#### **Premium Reports

- - Yearly forecast: $9.99
- - Full birth chart: $19.99
- - Compatibility report: $14.99

#### **AI Chat Credits

- - 5 free questions/month
- - $4.99 for 50 questions
- - Unlimited with subscription

## Next Steps

[ **

#### Kundli Generator

Birth chart visualization

](https://vedika.io/docs/sdk/kundli-generator) [ **

#### Panchang Widget

Daily calendar widget

](https://vedika.io/docs/sdk/panchang-widget)

---
_Full page (always current): [https://vedika.io/docs/sdk/horoscope-app](https://vedika.io/docs/sdk/horoscope-app) · [Vedika docs](https://vedika.io/docs)_
