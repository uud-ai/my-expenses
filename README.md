# Финансы — учёт расходов

Мобильное PWA-приложение для учёта повседневных расходов: один файл `index.html`, без сборщиков и фреймворков, с синхронизацией между устройствами через Supabase.

## Возможности

- Добавление/редактирование/удаление трат, разбивка по дням
- Категории расходов — свои дефолтные + можно добавлять и удалять свои
- Повторяющиеся расходы (подписки, аренда) — еженедельно/ежемесячно
- Поиск по названию, фильтр по категории
- История по дням/месяцам, управление подписками
- Аналитика: по категориям и дням месяца, месячный бюджет с прогресс-баром
- Экспорт в CSV
- Офлайн-режим: кэш в `localStorage`, очередь изменений, синхронизация при восстановлении сети
- Realtime-синхронизация между устройствами (Supabase Realtime)
- Тёмная/светлая тема (ручной переключатель)
- Устанавливается как приложение (PWA) на Android/iOS

## Стек

- Vanilla JS, HTML, [Tailwind CSS](https://tailwindcss.com/) (собран статически, без CDN)
- [Supabase](https://supabase.com/) — авторизация (email/password) и Postgres-таблицы для синхронизации
- Service Worker — офлайн-кэш оболочки приложения

## Запуск локально

```bash
npm install
npm run build:css   # собирает styles.css из tailwind-input.css
```

Затем открыть `index.html` через любой статический сервер (например `npx serve` или `python -m http.server`) — открывать напрямую как `file://` нельзя, Service Worker и часть запросов требуют `http(s)`.

## Supabase: настройка своего проекта

`SUPABASE_URL` и `SUPABASE_ANON_KEY` захардкожены в `index.html` (это публичный anon-ключ, предназначен для клиента — доступ к данным ограничивается Row Level Security). Чтобы запустить со своим проектом:

1. Создать проект на [supabase.com](https://supabase.com), включить Email-авторизацию.
2. Создать таблицы (структура ниже), включить RLS и политики «пользователь видит/пишет только свои строки» (`user_id = auth.uid()`).
3. В Authentication → URL Configuration добавить домен деплоя в Site URL / Redirect URLs (нужно для ссылки сброса пароля).
4. Подставить свои `SUPABASE_URL` / `SUPABASE_ANON_KEY` в `index.html`.

### Таблица `expenses`

| колонка | тип |
|---|---|
| `id` | uuid, primary key |
| `user_id` | uuid |
| `name` | text |
| `amount` | numeric |
| `category` | text |
| `created_at` | timestamptz |
| `updated_at` | timestamptz |
| `is_deleted` | boolean |
| `recurring_id` | uuid, nullable, FK → `recurring_expenses.id` |

### Таблица `recurring_expenses`

| колонка | тип |
|---|---|
| `id` | uuid, primary key |
| `user_id` | uuid |
| `name` | text |
| `amount` | numeric |
| `category` | text |
| `interval` | text (`weekly` \| `monthly`) |
| `next_due` | date (`YYYY-MM-DD`) |
| `is_active` | boolean |
| `created_at` | timestamptz |
| `updated_at` | timestamptz |

Месячный бюджет и список пользовательских категорий отдельных таблиц не требуют — хранятся в `user_metadata` сессии Supabase Auth (`monthly_budget`, `categories`) и синхронизируются вместе с ней.

## Деплой

Любой статический хостинг (сейчас — Vercel). Деплой — это просто отдача файлов репозитория как есть, сборки кроме `npm run build:css` не требуется.

## Структура репозитория

```
index.html          разметка + вся логика приложения
styles.css           собранный Tailwind (генерируется, не редактировать руками)
tailwind-input.css    входной файл для сборки Tailwind
tailwind.config.js    конфиг Tailwind
sw.js                 Service Worker (офлайн-кэш оболочки)
manifest.json         веб-манифест PWA
icon.svg              иконка приложения/фавикон
```
