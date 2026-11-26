# Wahrly — app & AI server

Product idea, Free/Pro and screens: see the [root README](../README.md).

This folder is the **Expo iPhone / Android app** plus the **Express AI backend** (`server/`).

> Wahrly should feel like an AI that happens to have a task list — not a task manager with AI bolted on.

## Stack

- Expo 57 + React Native + TypeScript + Expo Router
- Supabase (Auth + Postgres + RLS)
- Express AI backend (`POST /api/ai/chat`, `/api/ai/transcribe`)
- Groq → OpenAI → on-device local AI fallback
- Expo Notifications, Zustand, AsyncStorage
- StoreKit / Play via `expo-iap` (Wahrly Pro)
- Optional Sentry + PostHog ([`docs/observability.md`](docs/observability.md))

## Product surface

| Surface | Where |
|---------|--------|
| Native app | EAS / TestFlight / App Store — this folder |
| Landing + Privacy + Terms | [`../site/`](../site/) → [wahrly.com](https://wahrly.com) |
| AI API | Render (`rootDir: app/server`) — prod: `https://threeproxy-x9bi.onrender.com` |

Keys map: [`docs/keys.md`](docs/keys.md).  
Local Expo web is for UI checks only — **not** deployed.

## Quick start

```bash
cd app
cp .env.example .env
npm install --legacy-peer-deps
npm --prefix server install

# Terminal 1 — AI server
npm run server

# Terminal 2 — Expo
npm start
```

### Demo mode (no credentials)

Leave `.env` placeholders as-is.

1. Sign up with any email + password (6+ chars)
2. Complete onboarding
3. In Chat, try:
   - `Remind me to call Mom tomorrow at 7 PM`
   - `Завтра купить продукты и постирать`
   - `What do I need to do today?`
   - `I finished buying food`

Timed tasks schedule local notifications when permissions allow.

## Configure Supabase + AI

1. Create a Supabase project
2. Run `supabase/schema.sql` in the SQL editor
3. Enable Email auth (+ Apple / Google if needed — [`docs/social-auth-setup.md`](docs/social-auth-setup.md))
4. Get a Groq key: [console.groq.com/keys](https://console.groq.com/keys)
5. Fill `.env`:

```env
EXPO_PUBLIC_SUPABASE_URL=...
EXPO_PUBLIC_SUPABASE_ANON_KEY=...
EXPO_PUBLIC_API_URL=http://localhost:8787
GROQ_API_KEY=gsk_...
GROQ_MODEL=openai/gpt-oss-20b
# optional fallback:
# OPENAI_API_KEY=sk-...
SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...
```

> Never put `GROQ_API_KEY` / `OPENAI_API_KEY` in the mobile app. Only the server uses them.  
> Provider order: **Groq → OpenAI → local AI**.

## Native builds (EAS)

```bash
cd app
npx eas-cli login

npm run build:ios:testflight   # production + submit
npm run build:ios              # preview on registered device
npm run build:android
```

Deep link: `wahrly://` (Google Calendar OAuth returns into the app after Allow).

Details: [`docs/mobile-apps.md`](docs/mobile-apps.md) · TestFlight copy: [`docs/testflight-beta.md`](docs/testflight-beta.md) · App Store listing: [`docs/app-store-listing.md`](docs/app-store-listing.md).

## Deploy AI server (Render)

1. [dashboard.render.com](https://dashboard.render.com) → **New** → **Web Service** (or Blueprint from root `render.yaml`)
2. Settings:
   - **Root Directory:** `app/server`
   - **Build:** `npm ci --omit=dev` (or as in `render.yaml`)
   - **Start:** `npm start`
   - **Health:** `/health`
3. Secrets: `GROQ_API_KEY`, optional `OPENAI_API_KEY`, `SUPABASE_*`, `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`
4. Point the app at the service URL:

```env
EXPO_PUBLIC_API_URL=https://YOUR-SERVICE.onrender.com
```

```bash
curl https://YOUR-SERVICE.onrender.com/health
```

### Docker

```bash
cd app/server
docker build -t wahrly-ai .
docker run -p 8787:8787 -e GROQ_API_KEY=gsk_... wahrly-ai
```

## Project structure

```
app/
├── app/                 # Expo Router screens (tabs, auth, onboarding)
├── components/
├── constants/           # Theme, brand
├── lib/                 # store, i18n, pro gates, supabase, api
├── services/            # AI actions, IAP, local AI
├── server/              # Express: chat, transcribe, google, news
├── widgets/             # iOS Home Screen widget
├── supabase/schema.sql
├── docs/
├── .env.example
└── README.md
```

## AI contract

`POST /api/ai/chat`

```json
{
  "message": "Tomorrow at 6 PM remind me to buy groceries",
  "user_id": "...",
  "tasks": [],
  "history": []
}
```

```json
{
  "reply": "Done. I'll remind you tomorrow at 6 PM.",
  "actions": [
    {
      "type": "create_reminder",
      "title": "Buy groceries",
      "date": "2026-09-18",
      "time": "18:00"
    }
  ]
}
```

Supported actions: `create_task`, `update_task`, `complete_task`, `delete_task`, `create_reminder`.

## Free vs Pro (gates)

Aligned with `lib/pro.ts` and [`docs/unit-economics.md`](docs/unit-economics.md):

- **Free:** Tasks, Dates, on-device chat, Home, Settings, readonly Calendar
- **Pro:** Bills, Shopping, Life, News, Invest, Plan day, morning / evening / weekly rituals, reminders, widget, cloud chat (~400/day), voice (~200/day)

SKU: `com.wahrly.assistant.pro.monthly`.

## Google Calendar (readonly)

**Users:** Settings → Connect Google Calendar → Allow.  
**Owner setup:** [`docs/google-calendar-oauth.md`](docs/google-calendar-oauth.md).

## Voice

Mic → server transcription → same AI action pipeline. Pro-only (Whisper / STT cost). Free uses typed / on-device chat.

## Product boundaries

Wahrly does **not** read email, access banking, automate WhatsApp, or run ads. Calendar is readonly. Shopping / Bills / Life spaces are Pro features — not Free.
