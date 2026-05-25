# Инструкция: Загрузка данных в таблицах

> Документ описывает подход к подгрузке данных в таблицах со списком заявок (ипотека и другие продукты). Следовать этим правилам при любой работе с таблицами в проекте.

---

## Принцип: ленивая подгрузка (infinite scroll)

Мы используем **ленивую подгрузку** вместо пагинации. Пользователь скроллит вниз — данные подгружаются автоматически, без нажатия кнопок и смены страниц.

### Почему не пагинация

- Заявок может быть сотни и тысячи — листать страницами неудобно
- Пользователю нужен быстрый просмотр актуального списка, не навигация по конкретным номерам страниц
- Линейный сценарий работы: открыл список → нашёл нужную заявку → перешёл

---

## Параметры первичной загрузки

При открытии страницы загружаем **двойной объём** относительно видимой области:

```
видимая область ≈ 10–12 строк → загружаем 20–25 строк при открытии
```

**Зачем:** пользователь должен сразу иметь контент «за линией обреза» — это создаёт ощущение плавности и бесконечности списка.

```typescript
const INITIAL_PAGE_SIZE = 25;  // первичная загрузка
const NEXT_PAGE_SIZE = 20;     // каждая следующая порция
```

---

## Триггер подгрузки следующей порции

Используем `IntersectionObserver` на sentinel-элементе внизу списка. Подгружаем следующую порцию **заранее** — когда пользователь доскроллил до последних ~3 строк (не когда закончились все строки).

```typescript
// sentinel ставится за 3 строки до конца списка, не в самый конец
const PRELOAD_THRESHOLD = 3; // строк до конца

// пример: если загружено 25 строк, sentinel вешается на строку #22
```

```typescript
useEffect(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting && hasMore && !isLoadingMore) {
        loadNextPage();
      }
    },
    { threshold: 0.1 }
  );

  if (sentinelRef.current) observer.observe(sentinelRef.current);
  return () => observer.disconnect();
}, [hasMore, isLoadingMore]);
```

---

## Состояния загрузки

### 1. Первичная загрузка (скелетон)

Когда таблица открывается впервые или применяется фильтр/сортировка — показываем **скелетон-строки** вместо спиннера на весь экран.

Используем **`animate-pulse`** из Tailwind — он даёт плавный fade in/out через `opacity: 0.5` на 50% цикла. Это проще и производительнее, чем кастомный shimmer на `background-position`.

```tsx
// Скелетон строки таблицы — все плашки внутри animate-pulse
const SkeletonRow = () => (
  <tr className="animate-pulse">
    {/* Чекбокс */}
    <td className="px-3 py-3">
      <div className="size-4 rounded bg-gray-200" />
    </td>

    {/* ID + дата */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-20 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-12 rounded bg-gray-200" />
    </td>

    {/* Аватар */}
    <td className="px-3 py-3">
      <div className="size-8 rounded-full bg-gray-200" />
    </td>

    {/* ФИО + телефон */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-28 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-24 rounded bg-gray-200" />
    </td>

    {/* Статус-бейдж */}
    <td className="px-3 py-3">
      <div className="h-5 w-24 rounded-full bg-gray-200" />
    </td>

    {/* Объект */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-40 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-20 rounded bg-gray-200" />
    </td>

    {/* Сумма */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-28 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-16 rounded bg-gray-200" />
    </td>

    {/* Тип / программа */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-24 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-16 rounded bg-gray-200" />
    </td>

    {/* Обновлено */}
    <td className="px-3 py-3">
      <div className="h-2.5 w-20 rounded bg-gray-200" />
      <div className="mt-1.5 h-2 w-14 rounded bg-gray-200" />
    </td>
  </tr>
);

// Количество скелетон-строк ≈ кол-во строк на экране
const SKELETON_ROWS_COUNT = 12;
```

> **Почему `animate-pulse`, а не кастомный shimmer:**
> Tailwind реализует pulse через `opacity: 0.5` на середине цикла — это дешевле для GPU, чем анимация `background-position`. Браузер оптимизирует `opacity` через compositing без перерисовки.

**Поддержка `prefers-reduced-motion`** — Tailwind обрабатывает это автоматически через вариант `motion-safe:`:

```tsx
// Если нужно отключить анимацию для пользователей с motion-чувствительностью:
<tr className="motion-safe:animate-pulse">
```

По умолчанию `animate-pulse` срабатывает всегда. Вариант `motion-safe:animate-pulse` применит анимацию только если пользователь **не** указал `prefers-reduced-motion: reduce` в системных настройках.

### 2. Подгрузка следующей порции (спиннер внизу)

Когда пользователь доскроллил до порога — внизу таблицы появляется спиннер. Строки, которые уже загружены, остаются на месте, список не перестраивается.

**Выравнивание:** спиннер и текст выравниваются по левому краю колонки с чекбоксом (`pl-8`), не по центру.

**Начертание текста:** `font-normal`.

```tsx
{isLoadingMore && (
  <tr>
    <td colSpan={COLUMNS_COUNT}>
      <div className="flex items-center gap-x-2 py-4 pl-8 text-zinc-500">
        <svg
          className="size-4 animate-spin"
          viewBox="0 0 24 24"
          fill="none"
          aria-hidden="true"
        >
          <circle
            className="opacity-25"
            cx="12" cy="12" r="10"
            stroke="currentColor"
            strokeWidth="4"
          />
          <path
            className="opacity-75"
            fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
          />
        </svg>
        <span className="text-sm font-normal">Загружаем заявки...</span>
      </div>
    </td>
  </tr>
)}
```

Цвет задаётся через `text-zinc-500` на обёртке — SVG и текст наследуют его через `currentColor`.

### 3. Конец списка

Когда все данные загружены — показываем тихое сообщение с количеством загруженных записей в скобках. Не акцентируем на нём внимание.

**Выравнивание:** по левому краю колонки с чекбоксом (`pl-8`), не по центру.

**Формат:** `Загрузили все {entityLabel} ({totalCount})` — количество в скобках даёт пользователю контекст.

```tsx
{!hasMore && rows.length > 0 && (
  <tr>
    <td colSpan={COLUMNS_COUNT}>
      <div className="py-4 pl-8 text-sm text-zinc-500">
        Загрузили все {entityLabel} ({totalCount})
      </div>
    </td>
  </tr>
)}
```

`entityLabel` определяется внутри компонента таблицы на основе текущего маршрута — страница ничего не передаёт явно:

```typescript
import { useLocation } from 'react-router-dom';

const ENTITY_LABELS: Record<string, string> = {
  '/mortgage':          'заявки',
  '/document-packages': 'пакеты',
  '/insurance':         'полисы',
  '/registration':      'заявки',
};

const useEntityLabel = () => {
  const { pathname } = useLocation();
  const match = Object.keys(ENTITY_LABELS).find(path =>
    pathname.startsWith(path)
  );
  return match ? ENTITY_LABELS[match] : 'записи';
};
```

Фоллбэк `'записи'` — на случай нового раздела, который ещё не добавлен в `ENTITY_LABELS`. Не падает, но при добавлении нового маршрута в проект — добавьте и сюда.

### 4. Ошибка подгрузки

Если следующая порция не загрузилась — показываем строку с ошибкой и кнопкой повтора. Уже загруженные строки не трогаем.

```tsx
{loadError && (
  <tr className="load-error-row">
    <td colSpan={COLUMNS_COUNT}>
      <div className="load-error">
        <span>Не удалось загрузить заявки</span>
        <button onClick={retryLoadNextPage}>Повторить</button>
      </div>
    </td>
  </tr>
)}
```

---

## Градиентная «линия обреза» (fade-out снизу)

Чтобы пользователь видел, что список продолжается за нижней границей контейнера, добавляем **белый градиент-оверлей** поверх последних строк.

**Градиент показывается всегда**, пока есть отображаемые строки — независимо от того, загружены все данные или нет. Список может заканчиваться за нижней границей экрана даже после полной загрузки.

```tsx
<div className="table-container">
  <table>
    {/* ... */}
  </table>

  {/* Градиент показываем всегда пока есть строки */}
  {!isInitialLoading && rows.length > 0 && (
    <div className="table-fade-overlay" aria-hidden="true" />
  )}
</div>
```

```css
.table-container {
  position: relative;
  overflow-y: auto;
  /* задайте конкретную высоту или max-height под ваш лейаут */
}

.table-fade-overlay {
  position: sticky;
  bottom: 0;
  left: 0;
  right: 0;
  height: 80px;
  background: linear-gradient(
    to bottom,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.85) 60%,
    rgba(255, 255, 255, 1) 100%
  );
  pointer-events: none; /* клики проходят сквозь оверлей */
  z-index: 2;
}
```

> **Важно:** `pointer-events: none` обязателен — иначе оверлей будет перехватывать клики по строкам таблицы.

---

## Поведение при изменении фильтров и сортировки

При любом изменении фильтра, поиска или сортировки:

1. **Сбрасываем** накопленный список строк в `[]`
2. **Сбрасываем** курсор пагинации / номер страницы
3. **Показываем скелетон** (состояние первичной загрузки)
4. Загружаем первую порцию с новыми параметрами

```typescript
const resetAndReload = () => {
  setRows([]);
  setPage(0);
  setHasMore(true);
  setIsInitialLoading(true);
  loadFirstPage();
};

// вызываем при каждом изменении фильтров
useEffect(() => {
  resetAndReload();
}, [filters, sortOrder, searchQuery]);
```

---

## Структура состояния

```typescript
interface TableState {
  rows: ApplicationRow[];         // накопленный список строк
  totalCount: number;             // общее количество (из первого ответа API)
  isInitialLoading: boolean;      // первая загрузка / после смены фильтра → скелетон
  isLoadingMore: boolean;         // подгрузка следующей порции → спиннер снизу
  hasMore: boolean;               // есть ли ещё данные
  loadError: boolean;             // ошибка при подгрузке
  cursor: string | null;          // курсор для cursor-based пагинации
}
```

---

## Параметры API

Предпочтительно использовать **cursor-based пагинацию** (а не offset), так как список заявок может меняться пока пользователь скроллит.

```typescript
// Запрос
GET /api/applications?limit=20&cursor=eyJpZCI6MTI1Nzg0MH0=&...filters

// Ответ
{
  "items": [...],
  "total": 347,
  "nextCursor": "eyJpZCI6MTI1NzgyMH0=",  // null если данные закончились
  "hasMore": true
}
```

При использовании offset-пагинации: если новая заявка добавится в начало списка пока пользователь скроллит — строки сдвинутся и следующий offset-запрос вернёт дубликат. Cursor-based этого лишён.

---

## Краткая шпаргалка состояний

| Ситуация | Что показываем |
|---|---|
| Страница только открылась | Скелетон-строки (≈12 шт) |
| Применён фильтр / сортировка | Скелетон-строки (сброс списка) |
| Пользователь скроллит, данные грузятся | Спиннер в строке внизу таблицы |
| Данные есть (любые) | Градиентный fade-out снизу |
| Все данные загружены | «Загрузили все {entityLabel} (N)» |
| Ошибка подгрузки | Строка с сообщением + кнопка «Повторить» |
| Список пустой (нет результатов) | Empty state на месте таблицы |
