# PZ1: Микросервисы с HTTP

Этот проект демонстрирует разделение монолитного приложения на два микросервиса: Auth и Tasks, взаимодействующих через HTTP.

## Архитектура

- **Auth Service**: Обрабатывает аутентификацию, логин и проверку токенов.

- **Tasks Service**: Управляет задачами, требует аутентификацию для всех операций.

## Структура проекта

```
tech-ip-sem2/
  services/
    auth/
      cmd/auth/main.go
      internal/http/handlers.go
    tasks/
      cmd/tasks/main.go
      internal/http/handlers.go
      internal/http/middleware.go
      internal/service/tasks.go
  shared/
    middleware/
      requestid.go
      logging.go
  docs/
    pz17_api.md
  README.md
```

## Запуск сервисов

### Запуск Auth Service

```bash
cd services/auth
export AUTH_PORT=8081
go run ./cmd/auth
```

### Запуск Tasks Service

```bash
cd services/tasks
export TASKS_PORT=8082
export AUTH_BASE_URL=http://localhost:8081
go run ./cmd/tasks
```

## Тестирование

Curl-команды описаны в docs/pz17_api.md. Здесь они будут частично дублированы вместе со скриншотами.

#### Получить токен
```bash
Invoke-RestMethod -Method Post -Uri "http://localhost:8081/v1/auth/login" `
  -ContentType "application/json" `
  -Body '{"username":"student","password":"student"}' `
  -Headers @{ "X-Request-ID" = "req-001" }
```
![](misc/1%20Получить%20токен.png)

#### Проверить токен напрямую
```bash
Invoke-RestMethod -Method Get -Uri "http://localhost:8081/v1/auth/verify" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-002"
  }
```
![](misc/2%20Проверить%20токен%20напрямую.png)

#### Проверка недействительного токена
```bash
Invoke-RestMethod -Method Get -Uri "http://localhost:8081/v1/auth/verify" `
  -Headers @{
    "Authorization" = "Bearer bad-token"
    "X-Request-ID" = "req-002-bad"
  } -ErrorAction Stop
```
![](misc/3%20Проверка%20недействительного%20токена.png)

#### Создать задачу
```bash
Invoke-RestMethod -Method Post -Uri "http://localhost:8082/v1/tasks" `
  -ContentType "application/json" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-003"
  } `
  -Body '{
    "title":"Do PZ17",
    "description":"split services",
    "due_date":"2026-01-10"
  }'
```
![](misc/4%20Создать%20задачу.png)

#### Получить список задач
```bash
Invoke-RestMethod -Method Get -Uri "http://localhost:8082/v1/tasks" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-004"
  }
```
![](misc/5%20Получить%20список%20задач.png)

#### Получить конкретную задачу
```bash
Invoke-RestMethod -Method Get -Uri "http://localhost:8082/v1/tasks/t_1" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-005"
  }
```
![](misc/6%20Получить%20конкретную%20задачу.png)

#### Обновить задачу
```bash
Invoke-RestMethod -Method Patch -Uri "http://localhost:8082/v1/tasks/t_1" `
  -ContentType "application/json" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-006"
  } `
  -Body '{ "title":"Do PZ17 updated", "done":true }'
```
![](misc/7%20Обновить%20задачу.png)

#### Удалить задачу
```bash
Invoke-RestMethod -Method Delete -Uri "http://localhost:8082/v1/tasks/t_1" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-007"
  }
```
![](misc/8%20Удалить%20задачу.png)

#### Тест невалидного токена в Tasks
```bash
Invoke-RestMethod -Method Delete -Uri "http://localhost:8082/v1/tasks/t_1" `
  -Headers @{
    "Authorization" = "Bearer demo-token"
    "X-Request-ID" = "req-007"
  }
```
![](misc/9%20Тест%20невалидного%20токена%20в%20Tasks.png)

#### Тест без токена в Tasks
```bash
Invoke-RestMethod -Method Get -Uri "http://localhost:8082/v1/tasks" `
  -Headers @{ "X-Request-ID" = "req-009" } -ErrorAction Stop
```
![](misc/9%20Тест%20невалидного%20токена%20в%20Tasks.png)

## Границы

- Auth: Выдача и проверка токенов.

- Tasks: CRUD операции над задачами, с проверкой auth.