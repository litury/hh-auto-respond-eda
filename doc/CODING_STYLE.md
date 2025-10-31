# Стандарты кодирования

Этот документ описывает стандарты кодирования для проекта HH Auto Respond.

## Именование

### JavaScript/TypeScript файлы и директории

- **Файлы и папки:** `camelCase`
  - `userService.ts`, `orderHandler.ts`, `hhApiClient.ts`

- **Vue компоненты:** `PascalCase`
  - `UserProfile.vue`, `ResumeCard.vue`, `ApplicationList.vue`

### Переменные и функции

- **Параметры функций:** `_camelCase` (с префиксом подчеркивания)
  - `function getUser(_userId, _options) { ... }`

- **Переменные и функции:** `camelCase`
  - `const userName = "Ivan"`
  - `function fetchVacancies() { ... }`

- **Классы и интерфейсы:** `PascalCase`
  - `class UserService { ... }`
  - `interface IApplicationData { ... }`

- **Константы:** `UPPER_CASE`
  - `const MAX_APPLICATIONS_PER_DAY = 40`

- **Приватные переменные:** `p_camelCase` (префикс `p_`)
  - `const p_internalCache = new Map()`

- **Async функции:** суффикс `Async`
  - `async function fetchVacanciesAsync() { ... }`

## Структура проекта

### Модульная архитектура

Проект организован по модулям с четкой иерархией:

```
src/
├── modules/              # Функциональные модули
│   ├── auth/
│   │   ├── authService.ts
│   │   ├── parts/        # Внутренняя реализация (скрыта от других модулей)
│   │   │   └── tokenHelper.ts
│   │   └── index.ts      # Экспорты модуля
│   └── applications/
│       ├── applicationService.ts
│       └── index.ts
├── services/             # Общие сервисы
├── repositories/         # Работа с базой данных
└── shared/               # Переиспользуемые компоненты
```

### Правила структуры

1. **Папка `parts/`** — только внутри модулей
   - Используется для скрытия внутренней реализации
   - НЕ может находиться в корне проекта

2. **Ограничение файлов** — не более 7-10 файлов в одной папке
   - Если больше — создавайте подпапки

3. **Индексные файлы** — создавайте `index.ts` в каждой папке
   ```typescript
   // modules/auth/index.ts
   export { authService } from './authService'
   export { AuthModule } from './authModule'
   ```

## Стиль кода

### Vue.js 3

- Используйте `<script setup lang="ts">` для компонентов
- Composition API вместо Options API
- Pinia для управления состоянием
- Vue Router для маршрутизации

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useUserStore } from '@/stores/userStore'

const userStore = useUserStore()
const count = ref(0)
</script>
```

### TypeScript

- Избегайте `any`, если не абсолютно необходимо
- Используйте интерфейсы для структур данных
- Включайте строгую типизацию

```typescript
interface IApplication {
  id: string
  vacancyId: string
  status: 'sent' | 'viewed' | 'invited' | 'rejected'
  appliedAt: Date
}
```

### Async/Await

- Используйте `async/await` вместо промисов
- Называйте асинхронные функции с суффиксом `Async`

```typescript
async function fetchVacanciesAsync(_resumeId: string): Promise<IVacancy[]> {
  try {
    const response = await hhApiClient.get(`/resumes/${_resumeId}/similar_vacancies`)
    return response.data
  } catch (error) {
    console.error('Ошибка загрузки вакансий:', error)
    throw error
  }
}
```

### Обработка ошибок

- Используйте `try/catch` для API запросов
- Логируйте ошибки с контекстом

```typescript
try {
  await sendApplicationAsync(_vacancyId)
} catch (error) {
  console.error(`Ошибка отправки отклика на вакансию ${_vacancyId}:`, error)
  // Обработка ошибки
}
```

## Комментарии

- **Язык:** Русский
- **Цель:** Объяснять "почему", а не "что"
- **Стиль:** Краткие, без боковых комментариев

```typescript
// ✅ Хороший комментарий (объясняет причину)
// HH.ru разрешает максимум 200 откликов в день для предотвращения спама
const MAX_APPLICATIONS_PER_DAY = 200

// ❌ Плохой комментарий (описывает очевидное)
// Увеличиваем счетчик на 1
count++
```

Комментарии должны **уменьшать технический долг** и **упрощать поддержку** кода.

## Основные принципы

1. **Читаемость** — код должен быть понятен другим разработчикам
2. **Модульность** — разделяйте ответственность между модулями
3. **Типобезопасность** — используйте TypeScript строго
4. **Простота** — избегайте излишней сложности

---

Следование этим стандартам помогает поддерживать качество кода и упрощает командную разработку.
