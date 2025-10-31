# HH Auto Respond EDA - Веб-приложение для автооткликов на вакансии

Образовательный проект автоматической отправки откликов на вакансии HH.ru, построенный на событийно-управляемой архитектуре (Event-Driven Architecture). Стек: Vue.js 3 (frontend) + NestJS (backend) + PostgreSQL + Bull/BullMQ (очереди). В дальнейшем легко адаптируется под Telegram Mini App.

## 🚀 Быстрый старт

```bash
# 1. Установить зависимости и запустить всё
npm run setup

# 2. Открыть Prisma Studio для просмотра данных
npm run studio
```

Откроется http://localhost:5555 с GUI для работы с БД.

---

## 📦 Что включено

- ✅ PostgreSQL 16 в Docker
- ✅ Prisma ORM (схема + миграции)
- ✅ Моковые данные (2 пользователя, 3 резюме, 5 откликов)
- ✅ Brief.md — технический бриф проекта (по образцу лекции)
- ✅ Бэкапы/восстановление БД
- ✅ npm скрипты для управления

---

## 📋 npm команды

### База данных

| Команда | Описание |
|---------|----------|
| `npm run db:up` | Поднять PostgreSQL в Docker |
| `npm run db:down` | Остановить PostgreSQL |
| `npm run db:logs` | Показать логи PostgreSQL |
| `npm run db:status` | Статус контейнера |
| `npm run db:reset` | Полный сброс БД (удалить → создать → миграции → seed) |

### Prisma

| Команда | Описание |
|---------|----------|
| `npm run migrate` | Применить миграции (создаёт новую если схема изменилась) |
| `npm run seed` | Загрузить моковые данные |
| `npm run studio` | Открыть Prisma Studio (GUI для БД) |
| `npm run generate` | Сгенерировать Prisma Client |

### Бэкапы

| Команда | Описание |
|---------|----------|
| `npm run backup` | Создать бэкап БД → `backups/backup_TIMESTAMP.sql` |
| `npm run restore` | Восстановить БД из бэкапа (интерактивный выбор) |
| `npm run backup:list` | Список всех бэкапов |

### Утилиты

| Команда | Описание |
|---------|----------|
| `npm run setup` | Полная установка (зависимости + БД + миграции + seed) |
| `npm run clean` | Полная очистка (удалить БД, volumes, node_modules, миграции) |

---

## 🗄️ Схема базы данных

### Таблица `users` — пользователи Telegram

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID | Первичный ключ |
| telegram_id | BIGINT | Telegram ID (уникальный) |
| hh_user_id | VARCHAR | HH.ru User ID |
| email | VARCHAR | Email пользователя |
| full_name | VARCHAR | Полное имя |
| access_token | TEXT | OAuth access token HH.ru |
| refresh_token | TEXT | OAuth refresh token |
| token_expiry | TIMESTAMP | Срок действия токена |
| created_at | TIMESTAMP | Дата создания |
| updated_at | TIMESTAMP | Дата обновления |

### Таблица `resumes` — резюме из HH.ru

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID | Первичный ключ |
| hh_resume_id | VARCHAR | HH.ru Resume ID (уникальный) |
| title | VARCHAR | Название резюме |
| auto_respond_enabled | BOOLEAN | Включены ли автоотклики |
| user_id | UUID | Foreign Key → users.id |
| created_at | TIMESTAMP | Дата создания |
| updated_at | TIMESTAMP | Дата обновления |

### Таблица `applications` — отправленные отклики

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID | Первичный ключ |
| resume_id | UUID | Foreign Key → resumes.id |
| vacancy_id | VARCHAR | HH.ru Vacancy ID |
| vacancy_title | VARCHAR | Название вакансии |
| user_id | UUID | Foreign Key → users.id |
| status | VARCHAR | Статус (sent/viewed/invited/rejected/error) |
| applied_at | TIMESTAMP | Дата отклика |

---

## 🔧 Примеры использования

### Подключение к БД через DBeaver/TablePlus

```
Host:     localhost
Port:     5432
Database: hh_auto_respond_dev
User:     postgres
Password: postgres
```

### SQL запросы

```sql
-- Все пользователи с количеством откликов
SELECT
  u.id,
  u.full_name,
  u.email,
  COUNT(a.id) as applications_count
FROM users u
LEFT JOIN applications a ON u.id = a.user_id
GROUP BY u.id;

-- Резюме с включенными автооткликами
SELECT * FROM resumes WHERE auto_respond_enabled = true;

-- Отклики за последние 24 часа
SELECT
  a.*,
  r.title as resume_title,
  u.full_name as user_name
FROM applications a
JOIN resumes r ON a.resume_id = r.id
JOIN users u ON a.user_id = u.id
WHERE a.applied_at > NOW() - INTERVAL '24 hours'
ORDER BY a.applied_at DESC;
```

---

## 🔄 Работа с миграциями

### Изменить схему

1. Отредактировать `prisma/schema.prisma`
2. Запустить `npm run migrate`
3. Prisma создаст и применит миграцию

Пример - добавить поле:

```prisma
model Resume {
  // ... существующие поля
  skills Json?  // ← добавили новое поле для хранения навыков
}
```

```bash
npx prisma migrate dev --name add_skills_to_resume
# Создаст файл: prisma/migrations/TIMESTAMP_add_skills_to_resume/migration.sql
```

---

## 📚 Структура проекта

```
minimal-event-platform/
├── doc/
│   ├── brief.md                 # Технический бриф проекта (по образцу лекции)
│   ├── CONTRIBUTING.md          # Как участвовать в разработке
│   ├── CODING_STYLE.md          # Стандарты кодирования
│   └── GIT_WORKFLOW.md          # Правила работы с Git
├── docker/
│   └── docker-compose.yml       # PostgreSQL контейнер
├── prisma/
│   ├── schema.prisma            # Схема БД (User, Resume, Application)
│   ├── migrations/              # История миграций
│   └── seed.js                  # Моковые данные (2 пользователя, 3 резюме, 5 откликов)
├── backups/                     # SQL бэкапы
│   ├── .gitkeep
│   └── README.md
├── scripts/
│   ├── backup.js                # Скрипт создания бэкапа
│   └── restore.js               # Скрипт восстановления
├── package.json                 # npm команды
├── .gitignore
├── .env.example
└── README.md
```

---

## 🎯 Следующие шаги разработки

Текущая инфраструктура готова для добавления backend логики:

### Этап 1: Веб-приложение Vue.js + NestJS (MVP)
- [ ] **Frontend:** Vue.js 3 + TypeScript + Pinia + Tailwind CSS
- [ ] **Backend:** NestJS + REST API
- [ ] OAuth авторизация HH.ru (Passport.js)
- [ ] HH.ru API интеграция (похожие вакансии, отправка откликов)
- [ ] Bull/BullMQ очереди (vacancy.check, application.send)
- [ ] Repository Pattern для работы с БД
- [ ] Деплой: Frontend (Vercel/Netlify) + Backend (Railway/Render)

### Этап 2: Адаптация под Telegram Mini App
- [ ] Добавить Telegram SDK в HTML (~100 строк кода)
- [ ] Endpoint `/api/mini-app/init` для верификации
- [ ] JWT токены вместо сессий
- [ ] Telegram Bot (Grammy) для запуска Mini App
- [ ] Один backend обслуживает оба интерфейса

### Этап 3: Масштабирование
- [ ] Проверка подписки на канал (middleware)
- [ ] Kafka вместо Bull/BullMQ
- [ ] Микросервисная архитектура
- [ ] Аналитика и дашборд
- [ ] AI-генерация сопроводительных писем
- [ ] Telegram Payments (монетизация)

**См. [doc/brief.md](./doc/brief.md) для детального плана разработки.**

---

## 🛠️ Требования

- Docker Desktop
- Node.js 18+
- npm 9+

---

## 📝 Первые шаги после клонирования

```bash
# 1. Клонировать репозиторий
git clone <repository-url>
cd minimal-event-platform

# 2. Скопировать .env.example → .env
cp .env.example .env

# 3. Запустить полную установку
npm run setup

# 4. Открыть Prisma Studio
npm run studio
```

---

## 🔍 Частые сценарии

### Ежедневная работа

```bash
# Поднять БД
npm run db:up

# Просмотреть данные
npm run studio

# Сделать бэкап перед экспериментами
npm run backup

# Сбросить БД к начальному состоянию
npm run db:reset
```

### Изменение схемы БД

```bash
# 1. Отредактировать prisma/schema.prisma
# 2. Создать миграцию
npm run migrate

# 3. Проверить через Prisma Studio
npm run studio
```

### Работа с бэкапами

```bash
# Создать бэкап
npm run backup

# Посмотреть список бэкапов
npm run backup:list

# Восстановить из бэкапа
npm run restore
```

---

## 📋 Участие в разработке

Мы приветствуем любой вклад в проект! Ознакомьтесь с документацией:

- [doc/CONTRIBUTING.md](./doc/CONTRIBUTING.md) — как участвовать в разработке
- [doc/CODING_STYLE.md](./doc/CODING_STYLE.md) — стандарты кодирования
- [doc/GIT_WORKFLOW.md](./doc/GIT_WORKFLOW.md) — правила работы с Git

**Контакт:** Telegram [@divatoz](https://t.me/divatoz)

## 🎯 Основан на архитектуре

Проект создан для образовательных целей на основе лекций по проектированию веб-приложений. Форки и адаптации приветствуются!

**Архитектурные паттерны:**
- Событийно-управляемые системы (Event-Driven Architecture)
- Системы массового обслуживания (Queueing Theory)
- Repository Pattern + Dependency Injection
- Feature Flags для гибкости

**Стек:**
- Frontend: Vue.js 3 + TypeScript + Pinia + Tailwind CSS
- Backend: NestJS + Prisma ORM
- Database: PostgreSQL
- Queues: Bull/BullMQ (Redis)
- Infrastructure: Docker Compose
- Mini App: Telegram SDK (адаптация на этапе 2)

## 📄 Лицензия

MIT License — см. [LICENSE](./LICENSE)

---

Разработано [@divatoz](https://t.me/divatoz) для образовательных целей
