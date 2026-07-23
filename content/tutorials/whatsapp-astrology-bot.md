---
title: "How to Build WhatsApp Astrology Bot with Vedika API | Tutorial"
source: https://vedika.io/tutorials/whatsapp-astrology-bot
canonical: https://vedika.io/tutorials/whatsapp-astrology-bot
---

Tutorial

# Build a WhatsApp Astrology Bot

Create a WhatsApp chatbot that generates Kundalis, daily horoscopes, and compatibility reports using Vedika API.

## Prerequisites

- ✓ Node.js 18+ installed
- ✓ Vedika API key ([Get free key](https://vedika.io/dashboard))
- ✓ Twilio account with WhatsApp sandbox
- ✓ Basic JavaScript knowledge

## Step 1: Project Setup

```
mkdir whatsapp-astrology-bot
cd whatsapp-astrology-bot
npm init -y
npm install express twilio axios dotenv
```

## Step 2: Environment Variables

Create a `.env` file:
```
VEDIKA_API_KEY=your_vedika_api_key
TWILIO_ACCOUNT_SID=replace_with_your_account_sid
TWILIO_AUTH_TOKEN=replace_with_your_auth_token
PORT=3000
```

## Step 3: Create the Bot

Create `index.js`:
```
require('dotenv').config();
const express = require('express');
const twilio = require('twilio');
const axios = require('axios');

const app = express();
app.use(express.urlencoded({ extended: true }));

const VEDIKA_API = 'https://api.vedika.io';

// Vedika API helper
async function getHoroscope(sign) {
    const response = await axios.get(
        `${VEDIKA_API}/api/v1/horoscope/daily/${sign}`,
        { headers: { 'Authorization': `Bearer ${process.env.VEDIKA_API_KEY}` } }
    );
    return response.data;
}

async function getKundali(birthData) {
    const response = await axios.post(
        `${VEDIKA_API}/api/v1/kundali/basic`,
        birthData,
        { headers: { 'Authorization': `Bearer ${process.env.VEDIKA_API_KEY}` } }
    );
    return response.data;
}

// WhatsApp webhook
app.post('/webhook', async (req, res) => {
    const message = req.body.Body.toLowerCase();
    const from = req.body.From;

    let reply = '';

    if (message.includes('horoscope')) {
        // Extract zodiac sign from message
        const signs = ['aries','taurus','gemini','cancer','leo','virgo',
                       'libra','scorpio','sagittarius','capricorn','aquarius','pisces'];
        const sign = signs.find(s => message.includes(s)) || 'aries';

        const horoscope = await getHoroscope(sign);
        reply = `*${sign.toUpperCase()} Daily Horoscope*\n\n${horoscope.prediction}`;
    }
    else if (message.includes('kundali')) {
        reply = 'To generate your Kundali, please send:\n' +
                'kundali DD/MM/YYYY HH:MM City\n\n' +
                'Example: kundali 15/08/1990 14:30 Mumbai';
    }
    else {
        reply = 'Welcome to Astrology Bot!\n\n' +
                'Commands:\n' +
                '- horoscope [sign]\n' +
                '- kundali\n' +
                '- compatibility\n\n' +
                'Powered by Vedika API';
    }

    const twiml = new twilio.twiml.MessagingResponse();
    twiml.message(reply);
    res.type('text/xml').send(twiml.toString());
});

app.listen(process.env.PORT, () => {
    console.log(`Bot running on port ${process.env.PORT}`);
});
```

## Step 4: Configure Twilio

1. Go to Twilio Console > Messaging > WhatsApp Sandbox
2. Set webhook URL to `https://your-domain.com/webhook`
3. Join the sandbox by sending the code to the WhatsApp number

## Step 5: Deploy

```
# Deploy to Railway/Render/Heroku
# Or run locally with ngrok:
npm start
ngrok http 3000
```

## What You Can Build

### Daily Horoscopes

Send personalized daily predictions to users

### Kundali Generation

Generate birth charts from WhatsApp

### Match Compatibility

36 Guna matching for couples

### Panchang Updates

Daily Tithi, Nakshatra, Rahu Kaal

## Ready to Build?

Get your free API key and start building today.

[ Get API Key ](https://vedika.io/dashboard)

---
_Full page (always current): [https://vedika.io/tutorials/whatsapp-astrology-bot](https://vedika.io/tutorials/whatsapp-astrology-bot) · [Vedika docs](https://vedika.io/docs)_
