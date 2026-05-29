# Импорт Swagger → Postman Collection

## Ключевые правила

### 1. URL — строка с `{{baseUrl}}`
`url` в каждом request должен быть **строкой**, а не объектом:
```json
"url": "{{baseUrl}}/api/v1/endpoint"
```
Не использовать объект `{"base": "baseUrl", "path": [...]}` — Postman не ресолвит переменную.

### 2. Переменная `baseUrl` — в корневом `variable[]`
```json
"variable": [
  {
    "key": "baseUrl",
    "value": "http://host:port",
    "type": "string"
  },
  {
    "key": "token",
    "value": "",
    "type": "string"
  }
]
```
Без `auth` объекта — он ломает ресолт переменных. Если нужен auth — добавлять headers вручную в каждом request.

### 3. Группировка по контроллерам
Группировать endpoints по контроллерам (из path):
- `Auth` — `/api/v1/auth-*`, `/api/v1/sms-auth-*`
- `Health` — `/healthy`, `/ready`
- `Migration` — `/migration/*`
- `RADIUS` — `/radius/*`
- `SUDIR` — `/sudir/*`

### 4. Headers
- `Content-Type: application/json` — только для endpoints с body
- Auth headers — вручную в каждом request (не через `auth` объект)

### 5. Query params
- `type: "string"`, `value: ""`, `enabled: true/false`
- Описания из swagger

### 6. Body
```json
"body": {
  "mode": "json",
  "json": { ... }
}
```
Брать schema из `requestBody.content['application/json'].schema`

## Команды

```bash
# Скачать swagger.json
curl -s "http://host/swagger/v1/swagger.json" -o /tmp/swagger.json

# Проверить endpoints
python3 -c "
import json
with open('/tmp/swagger.json') as f:
    spec = json.load(f)
for path, methods in spec.get('paths', {}).items():
    print(path, '→', list(methods.keys()))
"
```
