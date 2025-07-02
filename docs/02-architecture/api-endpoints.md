# API Endpoints | Описание REST API <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT-License image"></a>

**Дата:** 2025-07-02
**Версия:** 0.5v
**Автор:** [MindlessMuse666](https://github.com/MindlessMuse666) ([Telegram](https://t.me/mindless_muse "Telegram"), [Email](mindlessmuse.666@gmail.com "Email"))

> _Связанные документы:_
>
> 1. [Архитектура системы](../02-architecture/architecture.md "Документ: архитектура системы")

---

## Оглавление
1. [Users](#1-users)
2. [Auth](#2-auth)
3. [Tasks](#3-tasks)
4. [Tags](#4-tags)
5. [Integrations](#5-integrations)

---

## 1. Users

### GET /api/users/me/
- **Назначение:** Получить информацию о текущем пользователе
- **Метод:** GET
- **Аутентификация:** JWT

#### Пример запроса
```http
GET /api/users/me/ HTTP/1.1
Authorization: Bearer <token>
```

#### Пример успешного ответа
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "is_active": true,
  "is_staff": false,
  "date_joined": "2024-07-01T12:00:00Z"
}
```

#### Возможные ошибки
- 401 Unauthorized — отсутствует или неверный токен
- 403 Forbidden — нет доступа

---

## 2. Auth

### POST /api/users/register/
- **Назначение:** Регистрация нового пользователя
- **Метод:** POST

#### Пример запроса
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

#### Пример успешного ответа
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com"
}
```

#### Возможные ошибки
- 400 Bad Request — невалидные данные, email уже занят

### POST /api/users/login/
- **Назначение:** Аутентификация пользователя и получение JWT-токена
- **Метод:** POST

#### Пример запроса
```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

#### Пример успешного ответа
```json
{
  "access": "<jwt_token>",
  "refresh": "<refresh_token>"
}
```

#### Возможные ошибки
- 400 Bad Request — неверный email или пароль

---

## 3. Tasks

### GET /api/tasks/
- **Назначение:** Получить список задач (фильтрация, сортировка, пагинация)
- **Метод:** GET
- **Аутентификация:** JWT

#### Пример запроса
```http
GET /api/tasks/?status=New&priority=High&page=1 HTTP/1.1
Authorization: Bearer <token>
```

#### Пример успешного ответа
```json
{
  "count": 2,
  "results": [
    {
      "id": 1,
      "title": "Сделать отчёт",
      "description": "Подготовить квартальный отчёт",
      "priority": "High",
      "deadline": "2024-07-10T18:00:00Z",
      "status": "New",
      "created_at": "2024-07-01T12:00:00Z",
      "updated_at": "2024-07-01T12:00:00Z",
      "tags": ["work", "urgent"]
    }
  ]
}
```

#### Возможные ошибки
- 401 Unauthorized — отсутствует или неверный токен

### POST /api/tasks/
- **Назначение:** Создать новую задачу
- **Метод:** POST
- **Аутентификация:** JWT

#### Пример запроса
```json
{
  "title": "Сделать отчёт",
  "description": "Подготовить квартальный отчёт",
  "priority": "High",
  "deadline": "2024-07-10T18:00:00Z",
  "status": "New",
  "tags": ["work", "urgent"]
}
```

#### Пример успешного ответа
```json
{
  "id": 1,
  "title": "Сделать отчёт",
  "status": "New"
}
```

#### Возможные ошибки
- 400 Bad Request — невалидные данные
- 401 Unauthorized

### GET /api/tasks/{id}/
- **Назначение:** Получить задачу по id
- **Метод:** GET
- **Аутентификация:** JWT

#### Пример запроса
```http
GET /api/tasks/1/ HTTP/1.1
Authorization: Bearer <token>
```

#### Пример успешного ответа
```json
{
  "id": 1,
  "title": "Сделать отчёт",
  "description": "Подготовить квартальный отчёт",
  "priority": "High",
  "deadline": "2024-07-10T18:00:00Z",
  "status": "New",
  "created_at": "2024-07-01T12:00:00Z",
  "updated_at": "2024-07-01T12:00:00Z",
  "tags": ["work", "urgent"]
}
```

#### Возможные ошибки
- 404 Not Found — задача не найдена
- 401 Unauthorized

### PUT /api/tasks/{id}/
- **Назначение:** Обновить задачу
- **Метод:** PUT
- **Аутентификация:** JWT

#### Пример запроса
```json
{
  "title": "Сделать отчёт (обновлено)",
  "status": "In Progress"
}
```

#### Пример успешного ответа
```json
{
  "id": 1,
  "title": "Сделать отчёт (обновлено)",
  "status": "In Progress"
}
```

#### Возможные ошибки
- 400 Bad Request — невалидные данные
- 404 Not Found
- 401 Unauthorized

### DELETE /api/tasks/{id}/
- **Назначение:** Удалить задачу
- **Метод:** DELETE
- **Аутентификация:** JWT

#### Пример запроса
```http
DELETE /api/tasks/1/ HTTP/1.1
Authorization: Bearer <token>
```

#### Пример успешного ответа
```json
{"detail": "Task deleted"}
```

#### Возможные ошибки
- 404 Not Found
- 401 Unauthorized

---

## 4. Tags

### GET /api/tags/
- **Назначение:** Получить список тегов
- **Метод:** GET
- **Аутентификация:** JWT

#### Пример запроса
```http
GET /api/tags/ HTTP/1.1
Authorization: Bearer <token>
```

#### Пример успешного ответа
```json
[
  {"id": 1, "name": "work"},
  {"id": 2, "name": "urgent"}
]
```

#### Возможные ошибки
- 401 Unauthorized

### POST /api/tags/
- **Назначение:** Создать новый тег
- **Метод:** POST
- **Аутентификация:** JWT

#### Пример запроса
```json
{"name": "important"}
```

#### Пример успешного ответа
```json
{"id": 3, "name": "important"}
```

#### Возможные ошибки
- 400 Bad Request — невалидные данные, тег уже существует
- 401 Unauthorized

---

## 5. Integrations

### POST /api/telegram/webhook/
- **Назначение:** Обработка входящих сообщений от Telegram Bot
- **Метод:** POST

#### Пример запроса
```json
{
  "message": {
    "chat": {"id": 123456},
    "text": "/start"
  }
}
```

#### Пример успешного ответа
```json
{"ok": true}
```

#### Возможные ошибки
- 400 Bad Request — невалидные данные

---

**Примечание:** Все примеры приведены для REST API. Для каждого endpoint рекомендуется использовать JWT-аутентификацию (кроме регистрации и логина). 