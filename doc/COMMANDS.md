# Commands Reference

Полный справочник npm команд для проекта HH Auto Respond EDA.

## База данных

| Команда | Описание |
|---------|----------|
| `npm run db:up` | Поднять PostgreSQL в Docker |
| `npm run db:down` | Остановить PostgreSQL |
| `npm run db:logs` | Показать логи PostgreSQL |
| `npm run db:status` | Статус контейнера |
| `npm run db:reset` | Полный сброс БД (удалить → создать → миграции → seed) |

### Примеры использования

```bash
# Запустить БД
npm run db:up

# Проверить статус
npm run db:status

# Посмотреть логи в реальном времени
npm run db:logs

# Остановить БД
npm run db:down
```

---

## Prisma

| Команда | Описание |
|---------|----------|
| `npm run migrate` | Применить миграции (создаёт новую если схема изменилась) |
| `npm run seed` | Загрузить моковые данные |
| `npm run studio` | Открыть Prisma Studio (GUI для БД) |
| `npm run generate` | Сгенерировать Prisma Client |

### Примеры использования

```bash
# Создать и применить миграцию после изменения schema.prisma
npm run migrate

# Открыть графический интерфейс БД
npm run studio

# Перезагрузить seed данные
npm run seed

# Сгенерировать Prisma Client (после изменения схемы)
npm run generate
```

---

## Бэкапы

| Команда | Описание |
|---------|----------|
| `npm run backup` | Создать бэкап БД → `backups/backup_TIMESTAMP.sql` |
| `npm run restore` | Восстановить БД из бэкапа (интерактивный выбор) |
| `npm run backup:list` | Список всех бэкапов |

### Примеры использования

```bash
# Создать бэкап перед экспериментами
npm run backup

# Посмотреть все доступные бэкапы
npm run backup:list

# Восстановить из бэкапа (выберете файл из списка)
npm run restore
```

---

## Утилиты

| Команда | Описание |
|---------|----------|
| `npm run setup` | Полная установка (зависимости + БД + миграции + seed) |
| `npm run clean` | Полная очистка (удалить БД, volumes, node_modules, миграции) |

### Примеры использования

```bash
# Первый запуск проекта после клонирования
npm run setup

# Полная очистка проекта (для переустановки с нуля)
npm run clean
```

---

## Типичные команды для разработки

### Ежедневная работа

```bash
# 1. Поднять БД
npm run db:up

# 2. Открыть Prisma Studio
npm run studio

# 3. (Работа с кодом...)

# 4. Остановить БД в конце дня
npm run db:down
```

### Изменение схемы БД

```bash
# 1. Отредактировать prisma/schema.prisma

# 2. Создать и применить миграцию
npm run migrate

# 3. Проверить в Prisma Studio
npm run studio
```

### Работа с бэкапами

```bash
# Перед экспериментами
npm run backup

# Если что-то пошло не так
npm run restore
```

### Полный сброс

```bash
# Сбросить БД к начальному состоянию
npm run db:reset

# Или полная очистка + переустановка
npm run clean
npm run setup
```

---

## Docker команды (прямые)

Если нужно работать напрямую с Docker:

```bash
# Логи PostgreSQL
docker logs -f hh-auto-respond-postgres

# Статус контейнера
docker ps --filter name=hh-auto-respond-postgres

# Подключиться к PostgreSQL CLI
docker exec -it hh-auto-respond-postgres psql -U postgres -d hh_auto_respond_dev

# Остановить и удалить всё
docker-compose -f docker/docker-compose.yml down -v
```

---

## Prisma CLI (прямые команды)

Если нужно работать напрямую с Prisma CLI:

```bash
# Создать миграцию с именем
npx prisma migrate dev --name add_new_field

# Применить миграции в production
npx prisma migrate deploy

# Сбросить БД (удалит все данные!)
npx prisma migrate reset

# Валидация схемы
npx prisma validate

# Форматирование schema.prisma
npx prisma format
```

---

## Переменные окружения

Команды используют переменные из `.env` файла:

```bash
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/hh_auto_respond_dev"
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=hh_auto_respond_dev
POSTGRES_PORT=5432
```

Перед первым запуском:

```bash
cp .env.example .env
```
