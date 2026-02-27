# Интеграция «Сбор данных» Mini App с LEADTEX

Инструкция по получению Telegram ID пользователя и отправке данных в LEADTEX.

---

## 1. Как Mini App получает Telegram ID

Когда пользователь открывает Mini App через Telegram бота, Telegram автоматически передаёт данные пользователя:

```javascript
const tg = window.Telegram?.WebApp;
const user = tg.initDataUnsafe?.user;
const telegram_id = user?.id;  // например: 123456789
```

**Telegram передаёт объект user:**
```javascript
{
    id: 123456789,           // Telegram ID пользователя
    first_name: "Иван",
    last_name: "Петров",
    username: "ivanpetrov",
    language_code: "ru"
}
```

---

## 2. Как данные отправляются в LEADTEX

При отправке формы Mini App делает POST-запрос на `/api/submit`, который проксирует данные в Leadteh webhook:

```javascript
// api/submit.js
const WEBHOOK_URL = 'https://rb257034.leadteh.ru/inner_webhook/fe02b6a2-14d6-46c3-836f-5fb8292d0e4f';

const payloadToLeadteh = {
    contact_by: 'telegram_id',
    search: String(telegram_id),
    variables: {
        customer_name: 'Иван Петров',
        customer_phone: '+79991234567',
        telegram_user_name: 'Иван Петров',
        telegram_id: 123456789,
        source: 'telegram-webapp-data-collection',
        first_name: 'Иван',
        last_name: 'Петров',
        registration_date: '2026-02-28',
        registration_source: 'telegram_mini_app'
    }
};
```

---

## 3. Настройка LEADTEX

### Шаг 1: Регистрация контакта

Пользователь **должен сначала написать боту**, чтобы LEADTEX создал контакт с его `telegram_id`.

```
Пользователь → Пишет /start боту → LEADTEX создаёт контакт с telegram_id
```

### Шаг 2: Inner Webhook

- **Аккаунт:** rb257034.leadteh.ru
- **Webhook ID:** fe02b6a2-14d6-46c3-836f-5fb8292d0e4f
- **Полный URL:** `https://rb257034.leadteh.ru/inner_webhook/fe02b6a2-14d6-46c3-836f-5fb8292d0e4f`

### Шаг 3: Сценарий обработки

В LEADTEX создайте сценарий:

```
Триггер: Inner Webhook (fe02b6a2-14d6-46c3-836f-5fb8292d0e4f)
    ↓
Действие: Обновить переменные контакта
    ↓
Действие: Отправить уведомление / тег / сообщение
```

---

## 4. Схема работы

```
┌───────────────────────────────────────────────────────┐
│  1. Пользователь открывает Mini App из Telegram бота  │
│     Telegram передаёт: { user: { id: 123456789 } }   │
└───────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────┐
│  2. Mini App (index.html)                             │
│     - Получает telegram_id                            │
│     - Пользователь заполняет форму                    │
│     - POST на /api/submit                             │
└───────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────┐
│  3. Vercel Serverless Function (api/submit.js)        │
│     - Валидирует данные                               │
│     - Перенаправляет на LEADTEX webhook               │
└───────────────────────────────────────────────────────┘
                          │
                          ▼
┌───────────────────────────────────────────────────────┐
│  4. LEADTEX (rb257034.leadteh.ru)                     │
│     - Ищет контакт по telegram_id                     │
│     - Записывает переменные в карточку                │
│     - Запускает сценарий                              │
└───────────────────────────────────────────────────────┘
```

---

## 5. Доступные переменные в LEADTEX

| Переменная | Описание | Пример |
|------------|----------|--------|
| `{{customer_name}}` | Имя и фамилия | Иван Петров |
| `{{customer_phone}}` | Телефон | +79991234567 |
| `{{telegram_user_name}}` | Имя из Telegram | Иван Петров |
| `{{telegram_id}}` | Telegram ID | 123456789 |
| `{{first_name}}` | Имя | Иван |
| `{{last_name}}` | Фамилия | Петров |
| `{{source}}` | Источник | telegram-webapp-data-collection |
| `{{registration_date}}` | Дата | 2026-02-28 |
| `{{registration_source}}` | Источник регистрации | telegram_mini_app |
| `{{ts}}` | Timestamp | ISO 8601 |

---

## 6. Отладка

### В консоли браузера (внутри Telegram)

```javascript
console.log(Telegram.WebApp.initDataUnsafe.user);
```

### Проверка API

```bash
curl -X POST https://YOUR-VERCEL-URL/api/submit \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Тест","lastName":"Тестов","phone":"+79991234567","telegram_id":123456789}'
```
