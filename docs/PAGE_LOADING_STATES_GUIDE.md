# Инструкция: Состояния загрузки страницы

> Описывает как вести себя контентной области при первой и повторной загрузке. При первичной загрузке сайдбар тоже скелетонится — набор модулей зависит от клиента и неизвестен до ответа сервера. При повторной загрузке сайдбар всегда живой.

---

## Два типа загрузки

| Тип | Когда | Что скелетонить |
|---|---|---|
| **Первичная** | Переход на страницу впервые | Заголовок, счётчик, поиск, фильтры, таблица |
| **Повторная** | Смена фильтра, поиска, сортировки | Только тело таблицы |

Разница принципиальная: при повторной загрузке пользователь уже ориентируется на странице — заголовок и фильтры дёргать не нужно.

---

## Задержка перед показом скелетона

Скелетон показываем **не сразу**, а только если данные не пришли за 500 мс. Это убирает мигание на быстром соединении — пользователь просто сразу видит готовый контент без промежуточного состояния.

```typescript
const SKELETON_DELAY_MS = 500;

const useDelayedSkeleton = (isLoading: boolean) => {
  const [showSkeleton, setShowSkeleton] = useState(false);

  useEffect(() => {
    if (!isLoading) {
      setShowSkeleton(false);
      return;
    }
    const timer = setTimeout(() => setShowSkeleton(true), SKELETON_DELAY_MS);
    return () => clearTimeout(timer);
  }, [isLoading]);

  return showSkeleton;
};
```

```tsx
const MortgagePage = () => {
  const { isInitialLoading, isReloading, ... } = usePageData();

  const showInitialSkeleton = useDelayedSkeleton(isInitialLoading);
  const showReloadSkeleton  = useDelayedSkeleton(isReloading);
};
```

Пока задержка не истекла — при первичной загрузке рендерим пустую область, при повторной — старые данные таблицы как есть.

---

## Скелетон сайдбара

Сайдбар скелетонится при первичной загрузке — набор пунктов меню зависит от подключённых модулей клиента и неизвестен до ответа сервера. При повторной загрузке (смена фильтров, поиск) сайдбар уже отрисован и не участвует.

```tsx
const SidebarSkeleton = () => (
  <div className="flex flex-col gap-1 p-3 animate-pulse">

    {/* Логотип */}
    <div className="mb-4 h-7 w-24 rounded bg-gray-200" />

    {/* Кнопка «Создать» */}
    <div className="mb-4 h-9 w-full rounded-md bg-gray-200" />

    {/* Пункты меню — имитируем 6–8 строк */}
    {Array.from({ length: 7 }).map((_, i) => (
      <div key={i} className="flex items-center gap-3 px-2 py-2">
        <div className="size-4 shrink-0 rounded bg-gray-200" />
        <div
          className="h-2.5 rounded bg-gray-200"
          style={{ width: `${55 + (i % 3) * 15}%` }}
        />
      </div>
    ))}

    {/* Разделитель + раздел «Инструменты» */}
    <div className="my-3 h-px bg-gray-100" />
    {Array.from({ length: 2 }).map((_, i) => (
      <div key={i} className="flex items-center gap-3 px-2 py-2">
        <div className="size-4 shrink-0 rounded bg-gray-200" />
        <div className="h-2.5 w-3/5 rounded bg-gray-200" />
      </div>
    ))}
  </div>
);
```

Ширины пунктов намеренно разные (`55%`, `70%`, `85%`) — одинаковые полоски выглядят механически и привлекают к себе лишнее внимание.

---

## Первичная загрузка

Срабатывает один раз — когда страница открывается и данных ещё нет вообще.

### Что скелетонить

```
┌─────────────────────────────────────────────────┐
│ ░░░░░░░░░░  ░░          ← заголовок + счётчик   │
│                                                  │
│ [ ░░░░░░░░░░░░░░░░░ ]  ░░░░░░░  ↓ ░░░░░░  [░░] │
│                         ← строка поиска/фильтров │
│ ─────────────────────────────────────────────── │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
│          ← строки таблицы                        │
└─────────────────────────────────────────────────┘
```

### Скелетон заголовка

Скелетонить только правую часть — иконку раздела и название не трогаем, они известны до загрузки. Скелетон нужен только для счётчика (`25`).

```tsx
const PageHeaderSkeleton = ({ title }: { title: string }) => (
  <div className="flex items-center gap-3">
    {/* Название известно сразу — не скелетоним */}
    <h1 className="text-xl font-semibold">{title}</h1>

    {/* Счётчик — скелетон, пока не пришёл totalCount */}
    <div className="animate-pulse">
      <div className="h-5 w-7 rounded bg-gray-200" />
    </div>
  </div>
);
```

Когда `totalCount` пришёл — заменяем на реальное число:

```tsx
{isInitialLoading
  ? <PageHeaderSkeleton title="Ипотека" />
  : <PageHeader title="Ипотека" count={totalCount} />
}
```

### Скелетон строки поиска и фильтров

```tsx
const FilterBarSkeleton = () => (
  <div className="flex items-center gap-3 animate-pulse">
    {/* Поле поиска */}
    <div className="h-8 w-72 rounded-md bg-gray-200" />

    {/* Кнопка фильтров */}
    <div className="h-8 w-24 rounded-md bg-gray-200" />

    {/* Сортировка */}
    <div className="h-8 w-32 rounded-md bg-gray-200" />

    {/* Кнопка скачать */}
    <div className="ml-auto h-8 w-8 rounded-md bg-gray-200" />
  </div>
);
```

---

## Повторная загрузка

Срабатывает при изменении фильтра, поиска или сортировки. Заголовок и фильтры остаются на месте — пользователь видит что он изменил и ждёт результата.

Вместо немедленной замены строк скелетоном — старые данные таблицы становятся **полупрозрачными**. Скелетон появляется только если данные не пришли за 500 мс (через `useDelayedSkeleton`).

```
до 500 мс — старые данные полупрозрачны:
┌─────────────────────────────────────────────────┐
│  Ипотека  347          ← живой                  │
│ [ Поиск по ID... ]  Фильтры  ↓ Сначала новые   │
│ ─────────────────────────────────────────────── │
│  ИП 1257847  НЕ  Новикова Е.Д.  ...  (opacity-50) │
│  ИП 1257846  НЕ  Новикова Е.Д.  ...  (opacity-50) │
│  ИП 1257845  ОД  Орлов Д.П.     ...  (opacity-50) │
└─────────────────────────────────────────────────┘

после 500 мс — скелетон:
┌─────────────────────────────────────────────────┐
│  Ипотека  347          ← живой                  │
│ [ Поиск по ID... ]  Фильтры  ↓ Сначала новые   │
│ ─────────────────────────────────────────────── │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
│ ░░░░  ░  ░░░░░░░░  ░░░░░░░  ░░░░░░░░░  ░░░░░░  │
└─────────────────────────────────────────────────┘
```

```tsx
{/* Таблица при повторной загрузке */}
{showReloadSkeleton
  ? <TableSkeleton rows={12} />
  : (
    <div className={isReloading ? 'opacity-50 pointer-events-none transition-opacity duration-150' : ''}>
      <DataTable ... />
    </div>
  )
}
```

`pointer-events-none` блокирует клики по строкам пока идёт загрузка. `transition-opacity duration-150` смягчает появление полупрозрачности — без резкого скачка.

---

## Структура состояния страницы

```typescript
interface PageState {
  isInitialLoading: boolean;  // true только при первом открытии
  isReloading: boolean;       // true при смене фильтров
}
```

Логика переключения:

```typescript
// Открытие страницы
setIsInitialLoading(true);
await loadFirstPage();
setIsInitialLoading(false);

// Смена фильтра / поиска / сортировки
setIsReloading(true);
resetTableRows();
await loadFirstPage();
setIsReloading(false);
```

---

## Разметка контентной области

```tsx
const MortgagePage = () => {
  const { isInitialLoading, isReloading, totalCount, ... } = usePageData();

  const showInitialSkeleton = useDelayedSkeleton(isInitialLoading);
  const showReloadSkeleton  = useDelayedSkeleton(isReloading);

  return (
    <div className="flex flex-col gap-4 p-6">

      {/* Заголовок — скелетон только при первичной загрузке */}
      {showInitialSkeleton
        ? <PageHeaderSkeleton title="Ипотека" />
        : <PageHeader title="Ипотека" count={totalCount} />
      }

      {/* Фильтры — скелетон только при первичной загрузке */}
      {showInitialSkeleton
        ? <FilterBarSkeleton />
        : <FilterBar ... />
      }

      {/* Таблица — скелетон при первичной, opacity при повторной */}
      {showInitialSkeleton || showReloadSkeleton
        ? <TableSkeleton rows={12} />
        : (
          <div className={isReloading
            ? 'opacity-50 pointer-events-none transition-opacity duration-150'
            : ''
          }>
            <DataTable ... />
          </div>
        )
      }

    </div>
  );
};
```

---

## Краткая шпаргалка

| Элемент | Первичная загрузка | Повторная загрузка |
|---|---|---|
| Сайдбар | Скелетон после 500 мс | Живой |
| Заголовок страницы | Скелетон после 500 мс (только счётчик) | Живой |
| Строка поиска / фильтров | Скелетон после 500 мс | Живой |
| Тело таблицы | Скелетон после 500 мс | `opacity-50` → скелетон после 500 мс |
