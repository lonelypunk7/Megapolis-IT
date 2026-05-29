# Megapolis-IT — Testing Infrastructure

## ⚡ Быстрый старт

### 1. Создайте `.env`

```bash
cp .env.example .env
```

Откройте `.env` и впишите свои `TESTY_LOGIN` и `TESTY_PASSWORD`.

### 2. Готово

Все скрипты используют `.env`.

---

## Setup

### `.env` файл

```bash
# 1. Скопируйте шаблон
cp .env.example .env

# 2. Или создайте вручную
cat > .env << 'EOF'
TESTY_URL=https://testy.megapolis-it.pro
TESTY_LOGIN=<ваш_логин>
TESTY_PASSWORD=<ваш_пароль>
EOF
```

**Где взять креды:** спросите у Леонида или в менеджере паролей.

**Важно:** `.env` уже в `.gitignore` — ваши креды не попадут в гит.

---

## Что здесь лежит

| Файл | Описание |
|---|---|
| `testy_mcp.py` | MCP сервер для TestY TMS (авторизация, создание/редактирование тест-кейсов) |
| `postman_collection.json` | Postman коллекция для Mega-wifi API |
| `testy_context.md` | Полный контекст TestY API — все endpoints, поля, типовые сценарии |
| `testy/case_rules.md` | Правила создания тест-кейсов в TestY |
| `.env` | Креды для TestY |
| `importSwaggerToPostman.md` | Как генерировать Postman коллекцию из Swagger |

## TestY TMS

### Создание тест-кейса

1. Проект: `22`
2. Suite Smoke: `341`
3. По правилам из `testy/case_rules.md`

**Пример создания через API:**
```bash
curl -s -X POST 'https://testy.megapolis-it.pro/api/v2/cases/' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Название теста",
    "project": 22,
    "suite": 341,
    "description": "Цель: проверить ...",
    "setup": "Предусловия: ...",
    "is_steps": true,
    "steps": [
      {"name": "Шаг 1", "scenario": "...", "expected": "...", "sort_order": 1},
      {"name": "Шаг 2", "scenario": "...", "expected": "...", "sort_order": 2}
    ],
    "labels": []
  }'
```

### Структура проекта TestY

```
Project 22 — ITM
├── Suite 341 — Smoke (критические E2E)
│   ├── 4557: Автооткрытие captive portal после подключения к SSID
│   ├── 4560: Открытие портала без ручного URL
│   ├── 4561: Доступность формы на целевых платформах
│   ├── 4562: Отображение всех 4 способов авторизации
│   ├── 4563: Успешная авторизация через ЕОС
│   ├── 4564: Успешная авторизация через MosID
│   ├── 4565: Успешная авторизация через YandexID
│   ├── 4566: Успешная авторизация через Госуслуги
│   ├── 4567: До авторизации доступ в интернет ограничен
│   ├── 4568: После авторизации доступ в интернет предоставляется
│   ├── 4569: Единый итог для всех 4 способов авторизации
│   └── 4571: Доступ к списку ресурсов без авторизации
├── Suite 342 — Autotests (автоматизированные)
│   ├── TC-CAP: Captive auth screen (5 кейсов)
│   ├── TC-CNS: Consent (согласие и документы) (5 кейсов)
│   ├── TC-PHN: Phone auth (SMS/OTP) (8 кейсов)
│   ├── TC-SSO: SSO (запланировано) (2 кейса)
│   └── TC-UT: Unit tests (9 кейсов)
├── Suite 338 — Клиентская часть
└── ...
```

### 4 способа авторизации

| Способ | Endpoint | Описание |
|---|---|---|
| ЕОС | `/api/v1/eos/auth` | Единая учётная запись |
| MosID | `/sudir/authorize` | mos.ru |
| YandexID | `/api/v1/yandex/auth` | Яндекс |
| Госуслуги | `/api/v1/gosuslugi/auth` | ЕСИА |
| SMS | `/api/v1/sms-auth/send-otp` + `/api/v1/sms-auth/verify-otp` | SMS-код |

### API endpoints TestY

| Endpoint | Method | Описание |
|---|---|---|
| `/api/token/` | POST | Login (JWT) |
| `/api/token/refresh/` | POST | Refresh token |
| `/api/v2/projects/` | GET | Список проектов |
| `/api/v2/cases/` | GET/POST | Тест-кейсы |
| `/api/v2/suites/` | GET/POST | Тест-сьюты |
| `/api/v2/testplans/` | GET/POST | Тест-планы |
| `/api/v2/results/` | GET/POST | Результаты тестов |
| `/api/v2/tests/` | GET/POST | Тесты |
| `/api/v2/comments/` | GET/POST | Комментарии |
| `/api/v2/users/` | GET/POST | Пользователи |
| `/api/v2/groups/` | GET/POST | Группы |
| `/api/v2/labels/` | GET | Лейблы |
| `/api/v2/statuses/` | GET | Статусы |
| `/api/v2/attachments/` | GET/POST | Аппачи |
| `/api/v2/notifications/` | GET | Уведомления |
| `/api/v2/custom-attributes/` | GET | Кастомные атрибуты |

## Mega-wifi API

### Endpoints

| Endpoint | Method | Описание |
|---|---|---|
| `/api/v1/auth-log/logs` | GET | Лог авторизаций |
| `/api/v1/sms-auth/send-otp` | POST | Отправить OTP |
| `/api/v1/sms-auth/verify-otp` | POST | Верификация OTP |
| `/radius/authorize` | POST | RADIUS авторизация |
| `/sudir/authorize` | POST | СУДИР авторизация |
| `/sudir/callback` | GET | СУДИР callback |
| `/healthy`, `/ready` | GET | Health checks |

## Структура проекта

```
/tmp/Megapolis-IT/
├── testy_mcp.py              # MCP сервер для TestY
├── postman_collection.json   # Postman коллекция
├── testy_context.md          # Контекст TestY API
├── testy/
│   └── case_rules.md         # Правила тест-кейсов
├── .env                      # Креды (не коммитить)
├── .env.example              # Шаблон
└── importSwaggerToPostman.md # Как генерировать Postman
```

## Быстрые команды

```bash
# Проверить авторизацию
curl -X POST https://testy.megapolis-it.pro/api/token/ \
  -H 'Content-Type: application/json' \
  -d '{"username": "leonidgalockin", "password": "ihS|4#Ba"}'

# Проверить health
curl -s http://mega-wifi.k8s.megapolis-it.pro/healthy
```
