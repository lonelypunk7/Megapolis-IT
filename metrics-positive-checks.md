# 15-min Metrics API — положительные проверки

`POST {{baseUrl}}{{metricsPath}}{{providerId}}`  
Basic Auth · тело: JSON-массив `[...]`

## Environment

| Переменная | Пример |
|------------|--------|
| `baseUrl` | `https://mega-wifi-backend.k8s.megapolis-it.pro` |
| `metricsPath` | `/api/external/v1/access-points/metrics/` |
| `providerId` | `nauka-svyaz` |
| `deviceId` | ID точки из БД |
| `deviceId2` | вторая точка (P-08) |
| `deviceMac` | `00:04:56:9e:da:30` |
| `authUser` / `authPass` | Basic Auth |

**Ожидание (все кейсы):** HTTP `200`, `status: "ok"`, `errors` пустой или нет.

---

## Сводная таблица

| ID | Название | Что проверяем |
|----|----------|---------------|
| P-01 | Smoke: availability=1 | 1 метрика, минимальный happy path |
| P-02 | Full: 7 метрик, 1 точка | Полный 15-мин срез |
| P-03 | availability=0 | Точка недоступна, значение валидно |
| P-04 | ts граница −1 час | Нижняя граница окна времени |
| P-05 | ts граница +15 мин | Верхняя граница окна времени |
| P-06 | ts=0 | Ноль допустим |
| P-07 | Нулевые числовые метрики | uptime/traffic/users = 0 |
| P-08 | 2 точки в 1 запросе | 2×7 метрик → 2 строки в CH |
| P-09 | ts из availability | Разные ts, в CH = ts availability |
| P-10 | ts min без availability | Нет availability, в CH = min(ts) |
| P-11 | Повторная отправка | Новый TS → новая строка в логе |

---

## P-01 — Smoke: availability = 1

```bash
TS=$(date +%s)
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' \
  -H 'Content-Type: application/json' \
  -d "[{
    \"id\": \"00:04:56:9e:da:30/availability/${TS}\",
    \"ts\": ${TS},
    \"device_id\": \"DEVICE_ID\",
    \"device_type\": \"ap\",
    \"sensor_type\": \"availability\",
    \"sensor_value\": 1
  }]"
```

---

## P-02 — Full: 7 метрик, 1 точка

```bash
TS=$(date +%s); MAC=00:04:56:9e:da:30; DEV=DEVICE_ID
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[
    {\"id\":\"${MAC}/availability/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1},
    {\"id\":\"${MAC}/uptime/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":413575},
    {\"id\":\"${MAC}/associated_users/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":11},
    {\"id\":\"${MAC}/traffic_tx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":1393305},
    {\"id\":\"${MAC}/traffic_rx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":2730419},
    {\"id\":\"${MAC}/traffic_tx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":580},
    {\"id\":\"${MAC}/traffic_rx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":1137}
  ]"
```

---

## P-03 — availability = 0

```bash
TS=$(date +%s)
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[{\"id\":\"00:04:56:9e:da:30/availability/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":0}]"
```

---

## P-04 — ts = now − 1 час

```bash
TS=$(($(date +%s) - 3600))
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[{\"id\":\"00:04:56:9e:da:30/availability/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1}]"
```

---

## P-05 — ts = now + 15 мин

```bash
TS=$(($(date +%s) + 900))
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[{\"id\":\"00:04:56:9e:da:30/availability/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1}]"
```

---

## P-06 — ts = 0

```bash
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[{\"id\":\"00:04:56:9e:da:30/availability/0\",\"ts\":0,\"device_id\":\"DEVICE_ID\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1}]"
```

---

## P-07 — нулевые числовые метрики

```bash
TS=$(date +%s); MAC=00:04:56:9e:da:30; DEV=DEVICE_ID
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[
    {\"id\":\"${MAC}/availability/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1},
    {\"id\":\"${MAC}/uptime/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":0},
    {\"id\":\"${MAC}/associated_users/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":0},
    {\"id\":\"${MAC}/traffic_tx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":0},
    {\"id\":\"${MAC}/traffic_rx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":0},
    {\"id\":\"${MAC}/traffic_tx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":0},
    {\"id\":\"${MAC}/traffic_rx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":0}
  ]"
```

---

## P-08 — 2 точки в одном запросе

```bash
TS=$(date +%s); MAC=00:04:56:9e:da:30
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[
    {\"id\":\"${MAC}/availability/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1},
    {\"id\":\"${MAC}/uptime/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":100},
    {\"id\":\"${MAC}/associated_users/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":5},
    {\"id\":\"${MAC}/traffic_tx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":1000},
    {\"id\":\"${MAC}/traffic_rx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":2000},
    {\"id\":\"${MAC}/traffic_tx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":100},
    {\"id\":\"${MAC}/traffic_rx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_1\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":200},
    {\"id\":\"${MAC}/availability/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1},
    {\"id\":\"${MAC}/uptime/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":200},
    {\"id\":\"${MAC}/associated_users/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":3},
    {\"id\":\"${MAC}/traffic_tx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":500},
    {\"id\":\"${MAC}/traffic_rx_volume/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":800},
    {\"id\":\"${MAC}/traffic_tx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":50},
    {\"id\":\"${MAC}/traffic_rx_speed/${TS}\",\"ts\":${TS},\"device_id\":\"DEVICE_ID_2\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":80}
  ]"
```

---

## P-09 — ts в CH = ts из availability

```bash
T1=$(date +%s); T2=$((T1-60)); T3=$((T1-120)); MAC=00:04:56:9e:da:30; DEV=DEVICE_ID
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[
    {\"id\":\"${MAC}/availability/${T1}\",\"ts\":${T1},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"availability\",\"sensor_value\":1},
    {\"id\":\"${MAC}/uptime/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":100},
    {\"id\":\"${MAC}/associated_users/${T3}\",\"ts\":${T3},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":2},
    {\"id\":\"${MAC}/traffic_tx_volume/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":100},
    {\"id\":\"${MAC}/traffic_rx_volume/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":200},
    {\"id\":\"${MAC}/traffic_tx_speed/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":10},
    {\"id\":\"${MAC}/traffic_rx_speed/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":20}
  ]"
```

**DB:** `ts` в ClickHouse = `T1`.

---

## P-10 — нет availability, ts = min

```bash
T1=$(date +%s); T2=$((T1-60)); T3=$((T1-300)); MAC=00:04:56:9e:da:30; DEV=DEVICE_ID
curl -sS -X POST "https://mega-wifi-backend.k8s.megapolis-it.pro/api/external/v1/access-points/metrics/nauka-svyaz" \
  -u 'USER:PASS' -H 'Content-Type: application/json' \
  -d "[
    {\"id\":\"${MAC}/uptime/${T1}\",\"ts\":${T1},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"uptime\",\"sensor_value\":100},
    {\"id\":\"${MAC}/associated_users/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"associated_users\",\"sensor_value\":2},
    {\"id\":\"${MAC}/traffic_tx_volume/${T3}\",\"ts\":${T3},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_volume\",\"sensor_value\":100},
    {\"id\":\"${MAC}/traffic_rx_volume/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_volume\",\"sensor_value\":200},
    {\"id\":\"${MAC}/traffic_tx_speed/${T1}\",\"ts\":${T1},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_tx_speed\",\"sensor_value\":10},
    {\"id\":\"${MAC}/traffic_rx_speed/${T2}\",\"ts\":${T2},\"device_id\":\"${DEV}\",\"device_type\":\"ap\",\"sensor_type\":\"traffic_rx_speed\",\"sensor_value\":20}
  ]"
```

**DB:** `ts` в ClickHouse = `T3`.

---

## P-11 — повторная отправка

Выполни P-02 дважды с разным `TS`. Ожидание: оба раза `200 ok`, в CH +2 строки.

---

## Postman — общий Tests (на папку)

```javascript
pm.test('200 OK', () => pm.response.to.have.status(200));
const body = pm.response.json();
pm.test('status ok', () => pm.expect(body.status).to.eql('ok'));
pm.test('provider', () => pm.expect(body.original_provider_id).to.eql(pm.environment.get('providerId')));
if (body.errors) pm.test('no errors', () => pm.expect(body.errors).to.be.empty);
```

## Postman — общий Pre-request (кроме P-04..P-06, P-09, P-10)

```javascript
pm.environment.set('ts', Math.floor(Date.now() / 1000));
```

Импорт коллекции: `metrics-positive-checks.postman_collection.json`
