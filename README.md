# Wahrly

**Your life, clearly.**  
Спокойный AI-помощник для повседневной жизни — задачи, даты, чат и тихие ритуалы в одном месте вместо пяти шумных приложений.

> Wahrly должен ощущаться как ИИ, у которого случился список дел — а не как таск-менеджер с прикрученным AI.

| | |
|--|--|
| Сайт | [wahrly.com](https://wahrly.com) |
| Privacy | [wahrly.com/privacy](https://wahrly.com/privacy) |
| Terms | [wahrly.com/terms](https://wahrly.com/terms) |
| iOS | `com.wahrly.assistant` · deep link `wahrly://` |
| API | Render · `app/server` |

---

## Идея

День современного человека размазан по заметкам, календарям, чатам и «я потом вспомню». Wahrly собирает это в одно спокойное поле:

1. **Сказал** — текстом или голосом: «завтра купить продукты», «напомни позвонить маме в 19:00».
2. **Увидел** — задача появляется на Home и в Tasks, с датой и временем.
3. **Закрыл день** — вечером Evening Clear помогает доделать, перенести и спокойно уйти в завтра.

Wahrly не читает почту, не лезет в банк и не пытается заменить всю жизнь. Он держит **ясность**: что сегодня важно, что ждёт ответа, какие даты нельзя пропустить — и как спокойно закончить вечер.

**Слоган:** *Close the day, clearly.*  
**Принцип:** *Plan the day. Keep the loops. Stay clear.*

---

## Для кого

Для людей, которым надоело жонглировать таск-листами, календарями и «умными» ассистентами, которые шумят больше, чем помогают. Wahrly — про спокойный день: добавил важное, увидел следующее, закрыл вечер.

---

## Что умеет приложение

### Free — ядро без облачного AI

| Возможность | Зачем |
|-------------|--------|
| **Tasks** | Дела с недельной лентой и подтверждением перед «готово» |
| **Dates** | Важные даты и мягкие напоминания |
| **Chat (on-device)** | Локальный AI: создаёт и двигает задачи без облака |
| **Home** | Таймлайн дня, open loops («я должен / жду»), календарная полоска |
| **Settings** | Имя, язык, Google Calendar (только чтение), выход |
| **Google Calendar** | Readonly: события дня на Home |

Free почти ничего не стоит в эксплуатации (~1–2¢ / MAU): без облачного LLM и без Whisper.

### Wahrly Pro — пространства, автопилот, облачный AI

| Возможность | Зачем |
|-------------|--------|
| **Plan day** | Автораскладка задач по свободным окнам |
| **Morning brief** | Утренний обзор дня (+ погода) |
| **Evening Clear** | Вечерний ритуал: закрыть день и подготовить завтра |
| **Weekly brief** | Понедельник — обзор недели |
| **Bills** | Счета и напоминания об оплате |
| **Shopping** | Список покупок |
| **Life / News / Invest** | Жизненные пространства, новости, котировки |
| **Cloud AI chat** | До ~400 запросов/день (Groq → OpenAI) |
| **Voice** | Диктовка → транскрипция → те же AI-действия (~200/день) |
| **Widget** | Home Screen: что дальше |

Подписка: `com.wahrly.assistant.pro.monthly` (базово **$9.99/мес**, цена региональная через App Store / Play).

---

## Экраны

| Экран | Роль |
|-------|------|
| **Home** | Приветствие, план сегодня, loops, composer |
| **Tasks** | Today / Tomorrow / Upcoming / Completed |
| **Chat** | Главный AI-интерфейс (local или cloud) |
| **Dates** | Дни рождения, праздники, важные даты |
| **Bills / Shopping / Life / News / Invest** | Pro-пространства |
| **Settings** | Профиль, ритуалы, календарь, Pro, язык |
| **Evening** | Ритуал закрытия дня |

Onboarding: welcome → имя → неделя → first value → ready.

---

## Как работает AI

Пользователь пишет или диктует. Сервер (или локальный fallback) отвечает текстом **и** структурированными действиями:

```json
{
  "reply": "Готово. Напомню завтра в 18:00.",
  "actions": [
    {
      "type": "create_reminder",
      "title": "Купить продукты",
      "date": "2026-09-18",
      "time": "18:00"
    }
  ]
}
```

Actions: `create_task`, `update_task`, `complete_task`, `delete_task`, `create_reminder`.

Приоритет провайдера: **Groq → OpenAI → встроенный local AI**.  
Ключи LLM живут **только на сервере**, никогда в мобильном приложении.

---

## Приватность и границы продукта

- Google Calendar — **только чтение**.
- **Нет** доступа к Gmail / почте.
- Нет банковских интеграций, WhatsApp-автоматизации и рекламного трекинга «ради продажи».
- Crash / analytics (Sentry, PostHog) — опционально, выключены без ключей.
- Политика и условия: [`app/docs/privacy-policy.md`](app/docs/privacy-policy.md), [`app/docs/terms-of-use.md`](app/docs/terms-of-use.md).

---

## Структура репозитория

```
wahrly/
├── app/                 # Expo iOS/Android + API (app/server)
│   ├── app/             # Экраны Expo Router
│   ├── components/
│   ├── lib/             # store, i18n, Pro-gates, supabase
│   ├── services/        # AI actions, IAP, local AI
│   ├── server/          # Express: chat, transcribe, google, news
│   ├── widgets/         # iOS Home Screen widget
│   ├── supabase/        # schema.sql
│   └── docs/            # App Store, ключи, OAuth, экономика
├── site/                # Лендинг + Privacy + Terms (GitHub Pages → wahrly.com)
├── render.yaml          # Blueprint AI-сервера
└── README.md            # этот файл
```

- **Приложение:** `cd app` → Expo / EAS  
- **Сайт:** `site/` → [wahrly.com](https://wahrly.com)  
- **API:** `app/server` на Render  

Подробный dev-гайд: [`app/README.md`](app/README.md).

---

## Стек

| Слой | Технологии |
|------|------------|
| Клиент | Expo 57 · React Native · TypeScript · Expo Router |
| Состояние | Zustand · AsyncStorage |
| Auth / DB | Supabase (Auth + Postgres + RLS) · Apple / Google sign-in |
| AI-сервер | Express · Groq / OpenAI / Whisper |
| Уведомления | Expo Notifications |
| Монетизация | StoreKit / Play Billing (`expo-iap`) |
| Сборка | EAS Build · TestFlight · App Store |
| Лендинг | Статика GitHub Pages (`site/`) |

---

## Быстрый старт

```bash
cd app
cp .env.example .env
npm install --legacy-peer-deps
npm --prefix server install

# Терминал 1 — AI-сервер
npm run server

# Терминал 2 — приложение
npm start
```

Без ключей работает demo: любой email + пароль (6+), onboarding, локальный Chat.

Сборка TestFlight:

```bash
cd app
npm run build:ios:testflight
```

Секреты и карта ключей: [`app/docs/keys.md`](app/docs/keys.md).  
Юнит-экономика Free/Pro: [`app/docs/unit-economics.md`](app/docs/unit-economics.md).  
Текст App Store: [`app/docs/app-store-listing.md`](app/docs/app-store-listing.md).

---

## Документация

| Документ | О чём |
|----------|--------|
| [`app/README.md`](app/README.md) | Dev setup, AI contract, Render |
| [`app/docs/app-store-listing.md`](app/docs/app-store-listing.md) | Листинг App Store |
| [`app/docs/unit-economics.md`](app/docs/unit-economics.md) | Free/Pro COGS |
| [`app/docs/keys.md`](app/docs/keys.md) | Где какие ключи |
| [`app/docs/google-calendar-oauth.md`](app/docs/google-calendar-oauth.md) | Readonly Calendar |
| [`app/docs/social-auth-setup.md`](app/docs/social-auth-setup.md) | Apple / Google |
| [`app/docs/mobile-apps.md`](app/docs/mobile-apps.md) | EAS / нативные сборки |
| [`app/docs/testflight-beta.md`](app/docs/testflight-beta.md) | Бета и скриншоты |
| [`app/docs/observability.md`](app/docs/observability.md) | Sentry + PostHog |

---

**Plan the day. Keep the loops. Stay clear.**
