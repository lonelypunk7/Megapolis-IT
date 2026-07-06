# Seed: точки доступа для покрытия позитивных тестов (ежесуточный API)

`POST /api/external/v1/access-points/nauka-svyaz`

**Важно:**
- Тело на стенде: `{ "importModels": [ ... ] }` (если 400 `importModels required` — оберни массив)
- В **одном** запросе только точки **одного** `original_provider_id` → два запроса: A (головной) и B (субоператор)
- `id` в daily API = внешний ID точки → его же используй как `device_id` в 15-min metrics
- `gk_name` = `У-25-21` (должен быть в `contracts`)
- Субоператор `enforta` должен быть в `operators` и привязан к `nauka-svyaz`

## Запрос A — головной оператор (10 точек)

Покрывает:

| Точка | Что покрывает |
|-------|----------------|
| TEST-AP-001 | baseline active, MAC `:`, metrics P-01/P-02/P-11 |
| TEST-AP-002 | вторая точка, metrics P-08 |
| TEST-AP-003 | standalone=true + nas_ip |
| TEST-AP-004 | MAC формат `-` (тире) |
| TEST-AP-005 | status temporarily_disabled |
| TEST-AP-006 | status maintenance |
| TEST-AP-007 | status decommissioned |
| TEST-AP-008 | category ВУЗ, location_id числовой |
| TEST-AP-009 | координаты граница lat=90, lon=180 |
| TEST-AP-010 | координаты граница lat=-90, lon=-180 |

## Запрос B — субоператор enforta (2 точки)

| Точка | Что покрывает |
|-------|----------------|
| TEST-AP-SUB-001 | original_provider_id ≠ provider_id |
| TEST-AP-SUB-002 | вторая точка субоператора |

Файлы: `daily-ap-seed-request-A.json`, `daily-ap-seed-request-B.json`
