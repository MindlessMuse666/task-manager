# System Architecture Document: Task Manager с аналитикой и интеграциями <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT-License image"></a>

**Дата:** 2025-07-01
**Версия:** 0.1v
**Автор:** [MindlessMuse666](https://github.com/MindlessMuse666) ([Telegram](https://t.me/mindless_muse "Telegram"), [Email](mindlessmuse.666@gmail.com "Email"))

**Ссылка на подробности vision & scope системы:** [VISION_AND_SCOPE.md](./VISION_AND_SCOPE.md)

## 1. Схема архитектуры системы

```mermaid
flowchart TD
    A("Frontend <br> (React/TS)") -- "HTTP" --> B("API Gateway")
    B -- "HTTP" --> C{"Backend <br> (Django REST Framework)"}
    C -- "ORM" --> D[("PostgreSQL <br> (основная БД)")]
    C -- "Celery" --> E[("Redis <br> (кеш/брокер сообщений)")]
    C -- "API" --> F("Telegram Bot (уведомления)")
    C -- "API" --> G("Google Calendar (синхронизация дедлайнов)")

    classDef default stroke:#333,stroke-width:2.5px
    classDef frontend fill:#FFC6FF
    classDef backend fill:#BDB2FF
    classDef gateway fill:#FFD6A5
    classDef postgres fill:#9BF6FF
    classDef redis fill:#FFADAD
    classDef telegram fill:#A0C4FF
    classDef google_calendar fill:#CAFFBF

    class A frontend
    class B gateway
    class C backend
    class D postgres
    class E redis
    class F telegram
    class G google_calendar
```

## 2. Описание компонентов и технологий

### 2.1. Frontend (React/TS)

- **Фреймворк:** React (TypeScript) - для создания интерактивного и модульного пользовательского интерфейса. TypeScript обеспечивает статическую типизацию, что повышает надежность и упрощает отладку.
- **UI Library:** Tailwind CSS - для быстрой и гибкой стилизации компонентов с использованием utility-first подхода. Рассмотрите использование библиотеки компонентов, например, Material UI или Ant Design, для ускорения разработки и обеспечения консистентного внешнего вида.
- **Графики:** Chart.js - для визуализации данных и создания графиков для дашбордов.
- **State Management:** Zustand - для управления состоянием приложения (более простой и легковесный, чем Redux Toolkit).
- **HTTP Client:** Axios или Fetch API - для выполнения HTTP-запросов к бэкенду.
- **UI Component Documentation:** Storybook - для документирования UI-компонентов и создания living style guide.

### 2.2. API Gateway

- **Технологии:** Django (может отдавать статику).
- **Функциональность:**
  - Маршрутизация запросов к бэкенду.
  - Аутентификация и авторизация (проверка JWT-токенов).
  - Ограничение скорости запросов (Rate Limiting) для защиты от атак.
  - CORS policies (ограничение доменов для фронтенда).
  - CSRF protection (используется "из коробки" Django REST Framework).

### 2.3. Backend (Django REST Framework)

- **Фреймворк:** Django REST Framework - для создания RESTful API. Django обеспечивает ORM (Object-Relational Mapper), что упрощает взаимодействие с базой данных.
- **Асинхронные задачи:** Celery - для выполнения асинхронных задач (например, отправка уведомлений) в фоновом режиме. Redis используется в качестве брокера сообщений для Celery.
- **Аутентификация:** JWT (JSON Web Tokens) - для безопасной аутентификации пользователей.
- **Сериализация:** Django REST Framework Serializers - для преобразования данных между форматами JSON и Python.
- **API Documentation:** OpenAPI (Swagger/ReDoc) - для документирования API и автоматической генерации клиентских библиотек.

### 2.4. Database (PostgreSQL)

- **Технологии:** PostgreSQL - надежная и масштабируемая реляционная база данных.
- **Индексы:** Добавлены индексы для часто запрашиваемых полей (например, user_id в таблице tasks) для повышения производительности запросов.
- **Схема данных (пример):**
  - **users:** id, username, email, password, is_active, is_staff, date_joined.
  - **tasks:** id, user_id, title, description, priority, deadline, status, created_at, updated_at.
  - **tags:** id, name.
  - **task_tags:** task_id, tag_id.

### 2.5. Redis (Cache/Message Broker)

- **Технологии:** Redis - быстрый in-memory data store.
- **Использование:**
  - Кэширование данных для дашбордов и часто запрашиваемых ресурсов.
  - Брокер сообщений для Celery (очередь задач).
  - Хранение сессий пользователей (опционально).
  - Кэширование токенов Google Calendar API.

### 2.6. Telegram Bot API

- **Технологии:** aiogram - асинхронная библиотека для создания Telegram ботов (может быть заменена на telegram-bot-api).
- **Функциональность:** Отправка уведомлений пользователям о новых задачах, приближающихся дедлайнах и изменении статуса задач.
- **Обработка ошибок:** В случае ошибки отправки уведомления (например, из-за лимитов API) - повторные попытки отправки с экспоненциальной задержкой (exponential backoff) и логирование ошибок.

### 2.7. Google Calendar API

- **Технологии:** Google API Client Library.
- **Функциональность:** Синхронизация дедлайнов задач с Google Calendar пользователя. Требует OAuth 2.0 для аутентификации и авторизации.
- **Кэширование токенов:** Кэширование токенов Google Calendar API в Redis для снижения нагрузки на API и ускорения работы.
- **Обработка недоступности API:** В случае недоступности Google Calendar API - кэшировать запросы и повторить синхронизацию через 5 минут.

### 3. API Endpoints (примеры)

### 3.1. Tasks

- **GET /api/tasks/:** Получение списка задач (с возможностью фильтрации, сортировки и пагинации).
- **POST /api/tasks/:** Создание новой задачи.
- **GET /api/tasks/{id}/:** Получение информации о конкретной задаче.
- **PUT /api/tasks/{id}/:** Обновление существующей задачи.
- **DELETE /api/tasks/{id}/:** Удаление задачи.

### 3.2. Users

- **POST /api/users/register/:** Регистрация нового пользователя.
- **POST /api/users/login/:** Аутентификация пользователя и получение JWT-токена.
- **GET /api/users/me/:** Получение информации о текущем пользователе (требуется JWT-токен).

### 3.3. Tags

- **GET /api/tags/:** Получение списка тегов.
- **POST /api/tags/:** Создание нового тега.

### 3.4. Telegram

- **POST /api/telegram/webhook/:** Обработка входящих сообщений от Telegram Bot (используется для интеграции с ботом).

### 4. Безопасность

- Все API endpoints должны быть защищены JWT-аутентификацией (кроме endpoints для регистрации и логина).
- Использовать HTTPS для всех соединений.
- Валидировать входные данные для предотвращения атак типа SQL Injection и Cross-Site Scripting (XSS).
- Регулярно обновлять зависимости для устранения известных уязвимостей.
- Rate limiting для защиты от DoS-атак.
- CORS policies (ограничение доменов для фронтенда).
- CSRF protection (используется "из коробки" Django REST Framework).

### 5. Деплой

- **Локально:** docker-compose up (PostgreSQL + Redis + Django).
```yaml
version: "3.9"
services:
 db:
  image: postgres:14
  volumes:
   - db_data:/var/lib/postgresql/data/
  environment:
   POSTGRES_USER: postgres
   POSTGRES_PASSWORD: postgres
   POSTGRES_DB: postgres
  ports:
   - "5432:5432"

 redis:
  image: redis:7
  ports:
   - "6379:6379"

 backend:
  build: ./backend
  command: sh -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
  volumes:
   - ./backend:/app
  ports:
   - "8000:8000"
  depends_on:
   - db
   - redis
  environment:
   DATABASE_URL: postgres://postgres:postgres@db:5432/postgres
   REDIS_URL: redis://redis:6379/0

volumes:
 db_data:
```
- **Production:** Docker + Kubernetes (AWS/GCP) - [Пример: deployment.yaml, service.yaml].

### 6. Дальнейшие шаги

- **Спроектировать схему базы данных:** Определить таблицы, поля, типы данных, ключи и связи между таблицами. (❌В РАЗРАБОТКЕ: [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md))
- **Определить API endpoints:** Полностью описать все API endpoints, включая методы, параметры запросов, форматы данных и коды ответов. (❌В РАЗРАБОТКЕ: [API_ENDPOINTS.md](./API_ENDPOINTS.md))
- **Разработать план реализации:** Разбить проект на небольшие задачи и определить порядок их выполнения. (❌В РАЗРАБОТКЕ: [PLAN_OF_IMPLEMENTATION.md](./PLAN_OF_IMPLEMENTATION.md))
- **Настроить окружение разработки:** Установить необходимые инструменты и библиотеки. (❌В РАЗРАБОТКЕ: [SETUP_ENVIRONMENT.md](./SETUP_ENVIRONMENT.md))
- **Начать разработку Frontend и Backend.** (❌В РАЗРАБОТКЕ)
