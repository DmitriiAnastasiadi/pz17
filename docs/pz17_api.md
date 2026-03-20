# Документация API для PZ17

## Auth Service

### POST /v1/auth/login

Эндпоинт для логина.

**Запрос:**

```json
{
  "username": "student",
  "password": "student"
}
```

**Ответ 200:**

```json
{
  "access_token": "demo-token",
  "token_type": "Bearer"
}
```

**Ошибки:**

- 400: Неверный запрос

- 401: Неверные учетные данные

### GET /v1/auth/verify

Проверка токена.

**Заголовки:**

- Authorization: Bearer <token>

- X-Request-ID: <id>

**Ответ 200:**

```json
{
  "valid": true,
  "subject": "student"
}
```

**Ответ 401:**

```json
{
  "valid": false,
  "error": "unauthorized"
}
```

## Tasks Service

Все эндпоинты требуют заголовок Authorization.

### POST /v1/tasks

Создать задачу.

**Запрос:**

```json
{
  "title": "Read lecture",
  "description": "Prepare notes",
  "due_date": "2026-01-10"
}
```

**Ответ 201:**

```json
{
  "id": "t_1",
  "title": "Read lecture",
  "description": "Prepare notes",
  "due_date": "2026-01-10",
  "done": false
}
```

### GET /v1/tasks

Список задач.

**Ответ 200:**

```json
[
  {"id":"t_1","title":"Read lecture","done":false}
]
```

### GET /v1/tasks/{id}

Получить задачу.

**Ответ 200:**

```json
{
  "id":"t_1",
  "title":"Read lecture",
  "description":"Prepare notes",
  "done": false
}
```

**Ошибки:**

- 404: Не найдено

### PATCH /v1/tasks/{id}

Обновить задачу.

**Запрос:**

```json
{
  "title": "Updated title",
  "done": true
}
```

**Ответ 200:** Обновленная задача

### DELETE /v1/tasks/{id}

Удалить задачу.

**Ответ 204**

## Переменные окружения

- AUTH_PORT=8081

- TASKS_PORT=8082

- AUTH_BASE_URL=http://localhost:8081

## Примеры curl

### Получить токен

```bash
curl -X POST http://localhost:8081/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"student","password":"student"}'
```

### Создать задачу

```bash
curl -X POST http://localhost:8082/v1/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer demo-token" \
  -d '{"title":"Do PZ17","description":"split services","due_date":"2026-01-10"}'
```