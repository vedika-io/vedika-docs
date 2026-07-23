---
title: "How to Build Telegram Astrology Bot with Vedika API | Tutorial"
source: https://vedika.io/tutorials/telegram-astrology-bot
canonical: https://vedika.io/tutorials/telegram-astrology-bot
---

Tutorial

# Build a Telegram Astrology Bot

Create a Telegram bot that provides horoscopes, Kundali generation, and astrological insights using Vedika API.

## Prerequisites

- ✓ Node.js 18+ installed
- ✓ Vedika API key ([Get free key](https://vedika.io/dashboard))
- ✓ Telegram Bot Token from @BotFather
- ✓ Basic JavaScript knowledge

## Step 1: Create Bot with BotFather

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` command
3. Choose a name (e.g., "Astrology Guru")
4. Choose a username (e.g., "astrology_guru_bot")
5. Save the API token you receive

## Step 2: Project Setup

```
mkdir telegram-astrology-bot
cd telegram-astrology-bot
npm init -y
npm install telegraf axios dotenv
```

## Step 3: Environment Variables

Create a `.env` file:
```
VEDIKA_API_KEY=your_vedika_api_key
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
```

## Step 4: Create the Bot

Create `bot.js`:
```
require('dotenv').config();
const { Telegraf, Markup } = require('telegraf');
const axios = require('axios');

const bot = new Telegraf(process.env.TELEGRAM_BOT_TOKEN);
const VEDIKA_API = 'https://api.vedika.io';
const API_KEY = process.env.VEDIKA_API_KEY;

// Vedika API helper
async function callVedika(endpoint, data = null) {
    const config = {
        headers: { 'Authorization': `Bearer ${API_KEY}` }
    };
    if (data) {
        return axios.post(`${VEDIKA_API}${endpoint}`, data, config);
    }
    return axios.get(`${VEDIKA_API}${endpoint}`, config);
}

// Start command
bot.start((ctx) => {
    ctx.reply(
        '🔮 Welcome to Astrology Bot!\n\n' +
        'I can help you with:\n' +
        '🔶 /horoscope - Daily horoscope\n' +
        '🔶 /kundali - Generate birth chart\n' +
        '🔶 /match - Compatibility check\n' +
        '🔶 /panchang - Today\'s Panchang\n\n' +
        'Powered by Vedika API',
        Markup.keyboard([
            ['♈️ Aries', '♉️ Taurus', '♊️ Gemini'],
            ['♋️ Cancer', '♌️ Leo', '♍️ Virgo'],
            ['♎️ Libra', '♏️ Scorpio', '♐️ Sagittarius'],
            ['♑️ Capricorn', '♒️ Aquarius', '♓️ Pisces']
        ]).resize()
    );
});

// Horoscope command
bot.command('horoscope', (ctx) => {
    ctx.reply('Select your zodiac sign:', Markup.inlineKeyboard([
        [Markup.button.callback('Aries', 'horoscope_aries'),
         Markup.button.callback('Taurus', 'horoscope_taurus'),
         Markup.button.callback('Gemini', 'horoscope_gemini')],
        [Markup.button.callback('Cancer', 'horoscope_cancer'),
         Markup.button.callback('Leo', 'horoscope_leo'),
         Markup.button.callback('Virgo', 'horoscope_virgo')],
        [Markup.button.callback('Libra', 'horoscope_libra'),
         Markup.button.callback('Scorpio', 'horoscope_scorpio'),
         Markup.button.callback('Sagittarius', 'horoscope_sagittarius')],
        [Markup.button.callback('Capricorn', 'horoscope_capricorn'),
         Markup.button.callback('Aquarius', 'horoscope_aquarius'),
         Markup.button.callback('Pisces', 'horoscope_pisces')]
    ]));
});

// Handle horoscope callbacks
bot.action(/horoscope_(.+)/, async (ctx) => {
    const sign = ctx.match[1];
    await ctx.answerCbQuery();
    await ctx.reply(`Fetching ${sign} horoscope...`);

    try {
        const response = await callVedika(`/api/v1/horoscope/daily/${sign}`);
        const data = response.data;
        ctx.reply(
            `*${sign.toUpperCase()} Daily Horoscope*\n\n` +
            `${data.prediction || data.horoscope}\n\n` +
            `Lucky Number: ${data.lucky_number || 'N/A'}\n` +
            `Lucky Color: ${data.lucky_color || 'N/A'}`,
            { parse_mode: 'Markdown' }
        );
    } catch (error) {
        ctx.reply('Sorry, could not fetch horoscope. Please try again.');
    }
});

// Panchang command
bot.command('panchang', async (ctx) => {
    try {
        const response = await callVedika('/api/v1/panchang/today');
        const data = response.data;
        ctx.reply(
            `*Today's Panchang*\n\n` +
            `Tithi: ${data.tithi || 'N/A'}\n` +
            `Nakshatra: ${data.nakshatra || 'N/A'}\n` +
            `Yoga: ${data.yoga || 'N/A'}\n` +
            `Karana: ${data.karana || 'N/A'}\n\n` +
            `Sunrise: ${data.sunrise || 'N/A'}\n` +
            `Sunset: ${data.sunset || 'N/A'}`,
            { parse_mode: 'Markdown' }
        );
    } catch (error) {
        ctx.reply('Sorry, could not fetch Panchang. Please try again.');
    }
});

// Launch bot
bot.launch();
console.log('Bot is running...');

// Graceful shutdown
process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));
```

## Step 5: Run the Bot

```
node bot.js
```

Open Telegram and search for your bot. Send /start to begin!

## Bot Features You Can Add

### Inline Mode

Let users get horoscopes in any chat

### Daily Notifications

Schedule daily horoscope delivery

### Kundali PDF

Generate and send Kundali as PDF

### AI Chat

Natural language astrology queries

## Ready to Build?

Get your free API key and start building today.

[ Get API Key ](https://vedika.io/dashboard)

---
_Full page (always current): [https://vedika.io/tutorials/telegram-astrology-bot](https://vedika.io/tutorials/telegram-astrology-bot) · [Vedika docs](https://vedika.io/docs)_
