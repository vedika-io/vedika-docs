---
title: "Build an Astrology App Tutorial 2026 | React + Python Guide"
source: https://vedika.io/guides/build-astrology-app-tutorial
canonical: https://vedika.io/guides/build-astrology-app-tutorial
---

Complete Tutorial • 4000+ Words • Beginner Friendly

# Build a Complete Astrology App

Step-by-Step Guide with React, Python Flask, and Mobile Integration

## What We\'re Building

We\'ll create a full-featured astrology app with these capabilities:
- Birth chart calculator with planetary positions
- Kundali matching for relationships
- Daily horoscope generator
- AI-powered astrology chatbot
- User authentication and profile management
- Mobile-responsive design

### Tech Stack

#### Frontend

- • React 18 / Next.js 14
- • TailwindCSS for styling
- • Axios for API calls
- • React Query for caching

#### Backend

- • Python Flask API
- • SQLite/PostgreSQL
- • JWT authentication
- • Vedika API integration

### Prerequisites

- Basic knowledge of JavaScript/React
- Basic Python experience
- Node.js 18+ and Python 3.8+ installed
- Code editor (VS Code recommended)
- Vedika API key (get free at [vedika.io/dashboard](https://vedika.io/dashboard))

## Step 1: Project Setup

### Create Project Structure

```
# Create main project directory
mkdir astrology-app
cd astrology-app

# Create frontend (React)
npx create-next-app@latest frontend
cd frontend
npm install axios react-query @tanstack/react-query

# Create backend (Flask)
cd ..
mkdir backend
cd backend
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install flask flask-cors flask-jwt-extended requests python-dotenv

```

### Environment Variables

Create `.env` files in both frontend and backend:

backend/.env:
```
VEDIKA_API_KEY=your_vedika_api_key_here
VEDIKA_BASE_URL=https://api.vedika.io/v1
SECRET_KEY=your_secret_key_for_jwt
DATABASE_URL=sqlite:///astrology.db

```

frontend/.env.local:
```
NEXT_PUBLIC_API_URL=http://localhost:5000

```

## Step 2: React Frontend

### Create Birth Chart Component

frontend/components/BirthChartCalculator.jsx:
```
import { useState } from \'react\';
import axios from \'axios\';

export default function BirthChartCalculator() {
    const [formData, setFormData] = useState({
        name: \'\',
        date: \'\',
        time: \'\',
        city: \'\'
    });
    const [chart, setChart] = useState(null);
    const [loading, setLoading] = useState(false);

    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);

        try {
            // Get coordinates from city name
            const geocode = await axios.get(
                `https://nominatim.openstreetmap.org/search?q=${formData.city}&format=json`
            );

            const location = geocode.data[0];

            // Call our backend API
            const response = await axios.post(
                `${process.env.NEXT_PUBLIC_API_URL}/api/birth-chart`,
                {
                    date: formData.date,
                    time: formData.time,
                    latitude: parseFloat(location.lat),
                    longitude: parseFloat(location.lon)
                }
            );

            setChart(response.data);
        } catch (error) {
            console.error(\'Error:\', error);
            alert(\'Failed to generate chart\');
        } finally {
            setLoading(false);
        }
    };

    return (

            Birth Chart Calculator

            {chart && (

                    Your Birth Chart

                            Planetary Positions

                                {Object.entries(chart.planets).map(([planet, data]) => (

                                        {planet}:
                                        {data.sign} ({data.degree.toFixed(2)}°)

                                ))}

                            Houses

                                {Object.entries(chart.houses).slice(0, 4).map(([house, sign]) => (

                                        House {house}:
                                        {sign}

                                ))}

                    {chart.yogas && chart.yogas.length > 0 && (

                            Active Yogas

                                {chart.yogas.map((yoga, idx) => (

                                        {yoga.name}
                                        {yoga.description}

                                ))}

                    )}

            )}

    );
}

```

### Create AI Chatbot Component

frontend/components/AstrologyChatbot.jsx:
```
import { useState, useRef, useEffect } from \'react\';
import axios from \'axios\';

export default function AstrologyChatbot({ birthData }) {
    const [messages, setMessages] = useState([
        { role: \'assistant\', content: \'Ask me anything about your birth chart!\' }
    ]);
    const [input, setInput] = useState(\'\');
    const [loading, setLoading] = useState(false);
    const messagesEnd = useRef(null);

    const handleSend = async () => {
        if (!input.trim()) return;

        const userMessage = { role: \'user\', content: input };
        setMessages([...messages, userMessage]);
        setInput(\'\');
        setLoading(true);

        try {
            const response = await axios.post(
                `${process.env.NEXT_PUBLIC_API_URL}/api/ai-chat`,
                {
                    birth_data: birthData,
                    question: input
                }
            );

            const aiMessage = {
                role: \'assistant\',
                content: response.data.answer
            };

            setMessages(prev => [...prev, aiMessage]);
        } catch (error) {
            console.error(\'Error:\', error);
            setMessages(prev => [...prev, {
                role: \'assistant\',
                content: \'Sorry, I encountered an error. Please try again.\'
            }]);
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        messagesEnd.current?.scrollIntoView({ behavior: \'smooth\' });
    }, [messages]);

    return (

            AI Astrology Assistant

                {messages.map((msg, idx) => (

                            {msg.content}

                ))}

                 setInput(e.target.value)}
                    onKeyPress={(e) => e.key === \'Enter\' && handleSend()}
                    placeholder="Ask: When will I get married?"
                    className="flex-1 border p-2 rounded"
                    disabled={loading}
                />

    );
}

```

## Step 3: Flask Backend

### Create Main API Server

backend/app.py:
```
from flask import Flask, request, jsonify
from flask_cors import CORS
import requests
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

VEDIKA_API_KEY = os.getenv(\'VEDIKA_API_KEY\')
VEDIKA_BASE_URL = os.getenv(\'VEDIKA_BASE_URL\')

@app.route(\'/api/birth-chart\', methods=[\'POST\'])
def birth_chart():
    """Generate birth chart"""
    data = request.json

    try:
        # Call Vedika API
        response = requests.post(
            f\'{VEDIKA_BASE_URL}/birth-chart\',
            json=data,
            headers={
                \'Authorization\': f\'Bearer {VEDIKA_API_KEY}\',
                \'Content-Type\': \'application/json\'
            }
        )

        response.raise_for_status()
        return jsonify(response.json()[\'data\'])

    except requests.exceptions.RequestException as e:
        return jsonify({\'error\': str(e)}), 500

@app.route(\'/api/kundali-matching\', methods=[\'POST\'])
def kundali_matching():
    """Match two kundalis"""
    data = request.json

    try:
        response = requests.post(
            f\'{VEDIKA_BASE_URL}/kundali-matching\',
            json=data,
            headers={
                \'Authorization\': f\'Bearer {VEDIKA_API_KEY}\',
                \'Content-Type\': \'application/json\'
            }
        )

        response.raise_for_status()
        return jsonify(response.json()[\'data\'])

    except requests.exceptions.RequestException as e:
        return jsonify({\'error\': str(e)}), 500

@app.route(\'/api/ai-chat\', methods=[\'POST\'])
def ai_chat():
    """AI chatbot for astrology queries"""
    data = request.json

    try:
        response = requests.post(
            f\'{VEDIKA_BASE_URL}/ai/chat\',
            json=data,
            headers={
                \'Authorization\': f\'Bearer {VEDIKA_API_KEY}\',
                \'Content-Type\': \'application/json\'
            }
        )

        response.raise_for_status()
        return jsonify(response.json()[\'data\'])

    except requests.exceptions.RequestException as e:
        return jsonify({\'error\': str(e)}), 500

@app.route(\'/api/daily-horoscope/\', methods=[\'GET\'])
def daily_horoscope(sign):
    """Get daily horoscope for zodiac sign"""
    try:
        response = requests.post(
            f\'{VEDIKA_BASE_URL}/horoscope/daily\',
            json={\'sign\': sign},
            headers={
                \'Authorization\': f\'Bearer {VEDIKA_API_KEY}\',
                \'Content-Type\': \'application/json\'
            }
        )

        response.raise_for_status()
        return jsonify(response.json()[\'data\'])

    except requests.exceptions.RequestException as e:
        return jsonify({\'error\': str(e)}), 500

if __name__ == \'__main__\':
```

## Step 4: Mobile Integration

### React Native Version

For mobile apps, use React Native with similar components:
```
npx react-native init AstrologyAppMobile
cd AstrologyAppMobile
npm install axios @react-navigation/native

```

### Flutter Alternative

```
// lib/services/vedika_api.dart
import \'package:http/http.dart\' as http;
import \'dart:convert\';

class VedikaAPI {
  final String apiKey;
  final String baseUrl = \'https://api.vedika.io/v1\';

  VedikaAPI(this.apiKey);

  Future> getBirthChart(Map birthData) async {
    final response = await http.post(
      Uri.parse(\'$baseUrl/birth-chart\'),
      headers: {
        \'Authorization\': \'Bearer $apiKey\',
        \'Content-Type\': \'application/json\'
      },
      body: jsonEncode(birthData)
    );

    if (response.statusCode == 200) {
      return jsonDecode(response.body)[\'data\'];
    } else {
      throw Exception(\'Failed to load birth chart\');
    }
  }
}

```

## Step 5: Deployment

### Deploy Frontend to Vercel

```
# Install Vercel CLI
npm i -g vercel

# Deploy
cd frontend
vercel --prod

```

### Deploy Backend to Heroku

```
# Create Procfile
echo "web: gunicorn app:app" > Procfile

# Install gunicorn
pip install gunicorn
pip freeze > requirements.txt

# Deploy to Heroku
heroku create astrology-api-backend
git init
git add .
git commit -m "Initial commit"
heroku git:remote -a astrology-api-backend
git push heroku main

```

### Production Best Practices

- Use environment variables for API keys
- Implement rate limiting
- Add caching layer (Redis)
- Set up monitoring (Sentry)
- Use CDN for static assets
- Implement user authentication
- Add analytics tracking

## Ready to Build Your App?

FREE Sandbox for development. Plans from $12/month

[Get API Key](https://vedika.io/dashboard) [View Full Docs](https://vedika.io/docs)

---
_Full page (always current): [https://vedika.io/guides/build-astrology-app-tutorial](https://vedika.io/guides/build-astrology-app-tutorial) · [Vedika docs](https://vedika.io/docs)_
