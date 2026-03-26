# FinTrack MCP Server - Полное Руководство

**Version 1.1.0**
FinTrack
February 2026

> **Примечание:**
> Этот документ предназначен для AI агентов и LLM при работе с MCP сервером FinTrack.
> Содержит детальные примеры использования всех 58 инструментов.

---

## Содержание

1. [Обзор](#1-обзор)
2. [Accounts (Счета)](#2-accounts-счета)
3. [Categories (Категории)](#3-categories-категории)
4. [Transactions (Транзакции)](#4-transactions-транзакции)
5. [Recurring Payments (Повторяющиеся платежи)](#5-recurring-payments-повторяющиеся-платежи)
6. [Tags (Теги)](#6-tags-теги)
7. [Projects (Проекты)](#7-projects-проекты)
8. [Metrics (Метрики)](#8-metrics-метрики)
9. [Indicators (Показатели)](#9-indicators-показатели)
10. [Manual Metrics (Ручные метрики)](#10-manual-metrics-ручные-метрики)
11. [Разработка новых Tools](#11-разработка-новых-tools)

---

## 1. Обзор

### Конфигурация

| Среда | URL |
|-------|-----|
| Продакшн | `https://api.equity.su/mcp` |
| Локальная | `http://fintrack.test/mcp` |

Авторизация через Sanctum Bearer Token.

### Паттерн Prepare/Create

**КРИТИЧЕСКИ ВАЖНО**: Перед созданием ЛЮБОЙ сущности:

1. Вызови `prepare-{entity}` с данными
2. Покажи пользователю preview из ответа
3. Спроси подтверждение
4. Только после "Да" вызови `create-{entity}`

**Пример диалога:**

```
User: Создай счет Тинькофф
AI: [вызывает prepare-account]

    Создать счет?
    - Название: Тинькофф
    - Тип: checking (Расчетный счет)
    - Валюта: RUB
    - Начальный баланс: 0.00 ₽

    Подтвердить?

User: Да
AI: [вызывает create-account]
    ✓ Счет "Тинькофф" создан успешно!
```

### Паттерн Update

Для обновления передаются только изменяемые поля — остальные не трогаются.

```
User: Переименуй счет 1 в "Т-Банк"
AI: [вызывает update-account с account_id: 1, name: "Т-Банк"]
    ✓ Счет обновлён!
```

### Паттерн Delete

Перед удалением — уточни у пользователя, т.к. это необратимо.

```
User: Удали транзакцию 42
AI: Уверены? Это действие необратимо.
User: Да
AI: [вызывает delete-transaction с transaction_id: 42]
    ✓ Транзакция удалена.
```

---

## 2. Accounts (Счета)

### list-accounts

Получить все счета пользователя.

**Параметры:** нет

**Пример ответа:**
```json
{
  "accounts": [
    {
      "id": 1,
      "name": "Сбербанк",
      "type": "checking",
      "group": "cash",
      "currency": "RUB",
      "balance": 15000.50,
      "balance_formatted": "15000.50"
    }
  ],
  "total": 1
}
```

### get-account

Получить детали счета.

**Параметры:**
- `id` (required, integer) - ID счета

### prepare-account

Валидация перед созданием.

**Параметры:**
- `name` (required, string) - Название счета
- `type` (required, string) - Тип: checking, savings, credit, cash, investment, loan, other
- `group` (required, string) - Группа: cash, credit, savings, investment, loan, other
- `currency_code` (required, string) - Код валюты: RUB, USD, EUR
- `color` (optional, string) - Цвет в HEX: #FF5733
- `icon` (optional, string) - Идентификатор иконки
- `bank_id` (optional, integer) - ID банка
- `initial_balance` (optional, number) - Начальный баланс в копейках

### create-account

Создать счет (после prepare-account).

Параметры те же что и prepare-account.

### update-account

Обновить счет. Передавай только изменяемые поля.

**Параметры:**
- `account_id` (required, integer) - ID счета
- `name` (optional, string) - Новое название
- `type` (optional, string) - Тип: checking, savings, credit, cash, investment, loan, other
- `group` (optional, string) - Группа: cash, credit, savings, investment, loan, other
- `color` (optional, string) - Цвет в HEX
- `icon` (optional, string) - Идентификатор иконки
- `bank_id` (optional, integer) - ID банка
- `institution` (optional, string) - Название финансовой организации
- `last_four_digits` (optional, string) - Последние 4 цифры карты
- `description` (optional, string) - Описание

### delete-account

Удалить счет.

**Параметры:**
- `account_id` (required, integer) - ID счета
- `delete_transactions` (optional, boolean) - Также удалить все транзакции счета (default: false)

---

## 3. Categories (Категории)

### list-categories

Получить все категории.

**Параметры:** нет

### get-category

Получить детали категории.

**Параметры:**
- `id` (required, integer) - ID категории

### prepare-category

Валидация перед созданием.

**Параметры:**
- `name` (required, string) - Название категории
- `type` (required, string) - Тип: expense, income, both
- `icon` (optional, string) - Идентификатор иконки
- `color` (optional, string) - Цвет в HEX
- `parent_id` (optional, integer) - ID родительской категории
- `is_active` (optional, boolean) - Активна ли категория

### create-category

Создать категорию (после prepare-category).

### update-category

Обновить категорию.

**Параметры:**
- `category_id` (required, integer) - ID категории
- `name` (optional, string) - Новое название
- `type` (optional, string) - Тип: expense, income, both
- `icon` (optional, string) - Идентификатор иконки
- `color` (optional, string) - Цвет в HEX
- `description` (optional, string) - Описание

### delete-category

Удалить категорию. Нельзя удалить если есть транзакции или подкатегории.

**Параметры:**
- `category_id` (required, integer) - ID категории

---

## 4. Transactions (Транзакции)

### list-transactions

Получить транзакции с фильтрами.

**Параметры (все optional):**
- `from_date` (string) - Фильтр: дата начала (YYYY-MM-DD)
- `to_date` (string) - Фильтр: дата конца (YYYY-MM-DD)
- `type` (string) - Фильтр: тип (`expense` или `income`)
- `account_id` (integer) - Фильтр: ID счета
- `category_id` (integer) - Фильтр: ID категории
- `limit` (integer) - Результатов на странице (default: 50, max: 200)

**Пример ответа:**
```json
{
  "transactions": [...],
  "current_page": 1,
  "per_page": 50,
  "total": 142
}
```

### get-transaction

Получить детали транзакции.

**Параметры:**
- `id` (required, integer) - ID транзакции

### prepare-transaction

Валидация перед созданием.

**Параметры:**
- `account_id` (required, integer) - ID счета
- `amount` (required, number) - Сумма в копейках (10000 = 100.00)
- `type` (required, string) - Тип: expense или income
- `description` (optional, string) - Описание
- `category_id` (optional, integer) - ID категории
- `transaction_date` (optional, string) - Дата YYYY-MM-DD
- `notes` (optional, string) - Заметки
- `tag_ids` (optional, array) - Массив ID тегов
- `is_excluded` (optional, boolean) - Исключить из статистики

### create-transaction

Создать транзакцию (после prepare-transaction).

**Пример создания расхода:**
```
1. prepare-transaction:
   - account_id: 1
   - amount: 150000  (1500.00 RUB)
   - type: "expense"
   - description: "Продукты"
   - category_id: 5

2. [показать preview, получить подтверждение]

3. create-transaction с теми же данными
```

### update-transaction

Обновить транзакцию. Передавай только изменяемые поля.

**Параметры:**
- `transaction_id` (required, integer) - ID транзакции
- `amount` (optional, number) - Новая сумма в копейках
- `type` (optional, string) - Тип: expense или income
- `description` (optional, string) - Описание
- `category_id` (optional, integer) - ID категории
- `transaction_date` (optional, string) - Дата YYYY-MM-DD
- `notes` (optional, string) - Заметки
- `tag_ids` (optional, array) - Массив ID тегов (заменяет все теги)
- `is_excluded` (optional, boolean) - Исключить из статистики

### delete-transaction

Удалить транзакцию.

**Параметры:**
- `transaction_id` (required, integer) - ID транзакции

---

## 5. Recurring Payments (Повторяющиеся платежи)

### list-recurrings

Получить все повторяющиеся платежи.

**Параметры:** нет

### get-recurring

Получить детали платежа.

**Параметры:**
- `id` (required, integer) - ID платежа

### prepare-recurring

Валидация перед созданием.

**Параметры:**
- `name` (required, string) - Название платежа
- `amount` (required, number) - Сумма в копейках
- `type` (required, string) - Тип: expense, income, transfer
- `starts_at` (required, string) - Дата начала YYYY-MM-DD
- `description` (optional, string) - Описание
- `wallet_id` (optional, integer) - ID счета-источника
- `target_wallet_id` (optional, integer) - ID счета-получателя (для transfer)
- `category_id` (optional, integer) - ID категории
- `tag_ids` (optional, array) - Массив ID тегов
- `rrule` (optional, string) - Правило повторения RFC 5545
- `ends_at` (optional, string) - Дата окончания
- `is_active` (optional, boolean) - Активен ли платеж
- `auto_execute` (optional, boolean) - Автовыполнение
- `notify_before` (optional, integer) - Уведомить за N дней

### create-recurring

Создать повторяющийся платеж (после prepare-recurring).

**Примеры RRULE:**
- Ежемесячно 15 числа: `FREQ=MONTHLY;BYMONTHDAY=15`
- Каждую неделю: `FREQ=WEEKLY;INTERVAL=1`
- Каждые 2 недели: `FREQ=WEEKLY;INTERVAL=2`
- Каждый день: `FREQ=DAILY;INTERVAL=1`
- Первый понедельник месяца: `FREQ=MONTHLY;BYDAY=1MO`

### update-recurring

Обновить повторяющийся платеж.

**Параметры:**
- `recurring_id` (required, integer) - ID платежа
- `name` (optional, string) - Новое название
- `description` (optional, string) - Описание
- `amount` (optional, number) - Новая сумма в копейках
- `type` (optional, string) - Тип: expense, income, transfer
- `wallet_id` (optional, integer) - ID счета-источника
- `category_id` (optional, integer) - ID категории
- `rrule` (optional, string) - Правило повторения
- `starts_at` (optional, string) - Дата начала YYYY-MM-DD
- `ends_at` (optional, string) - Дата окончания
- `is_active` (optional, boolean) - Активен ли платеж

### delete-recurring

Удалить повторяющийся платеж.

**Параметры:**
- `recurring_id` (required, integer) - ID платежа

---

## 6. Tags (Теги)

### list-tags

Получить все теги.

**Параметры:** нет

### get-tag

Получить детали тега.

**Параметры:**
- `id` (required, integer) - ID тега

### prepare-tag

Валидация перед созданием.

**Параметры:**
- `name` (required, string) - Название тега
- `color` (optional, string) - Цвет в HEX (по умолчанию #3B82F6)

### create-tag

Создать тег (после prepare-tag).

### update-tag

Обновить тег.

**Параметры:**
- `tag_id` (required, integer) - ID тега
- `name` (optional, string) - Новое название
- `color` (optional, string) - Новый цвет в HEX

### delete-tag

Удалить тег.

**Параметры:**
- `tag_id` (required, integer) - ID тега

---

## 7. Projects (Проекты)

### list-projects

Получить все проекты пользователя.

**Параметры:** нет

### get-project

Получить детали проекта.

**Параметры:**
- `id` (required, integer) - ID проекта

### prepare-project

Валидация перед созданием.

**Параметры:**
- `name` (required, string) - Название проекта
- `description` (optional, string) - Описание
- `color` (optional, string) - Цвет в HEX: #FF5733
- `currency` (optional, string) - ISO 4217: RUB, USD, EUR
- `budget` (optional, number) - Бюджет
- `start_date` (optional, string) - Дата начала YYYY-MM-DD
- `end_date` (optional, string) - Дата окончания YYYY-MM-DD
- `parent_id` (optional, integer) - ID родительского проекта (подпроект)

### create-project

Создать проект (после prepare-project).

### update-project

Обновить проект.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `name` (optional, string)
- `description` (optional, string)
- `color` (optional, string)
- `currency` (optional, string)
- `budget` (optional, number)
- `start_date` (optional, string)
- `end_date` (optional, string)
- `status` (optional, string) - статус проекта

### delete-project

Удалить проект.

**Параметры:**
- `project_id` (required, integer) - ID проекта

---

## 8. Metrics (Метрики)

Метрики — автоматически вычисляемые финансовые показатели проекта по периодам. Автометрики (income, expense, balance) материализуются из транзакций и запланированных платежей. Вычисляемые метрики (indicators) рассчитываются по формулам.

### get-metrics-range

Получить все значения метрик проекта за диапазон дат.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `from` (required, string) - Дата начала (YYYY-MM-DD)
- `to` (required, string) - Дата окончания (YYYY-MM-DD)
- `scenario` (optional, string) - Сценарий (default: baseline)
- `compute` (optional, boolean) - Пересчитать вычисляемые метрики перед чтением (default: true)

**Пример использования:**
```
get-metrics-range:
  project_id: 1
  from: "2026-01-01"
  to: "2026-03-31"
  scenario: "baseline"
  compute: true
```

### get-metrics-monthly

Получить помесячный тренд для конкретной метрики, с возможностью сравнения сценариев.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `key` (required, string) - Суффикс ключа метрики (например: "income", "expense"). Автоматически превращается в `p:{project_id}:{key}`
- `months` (optional, integer) - Количество месяцев назад (default: 12, max: 60)
- `scenarios` (optional, array) - Массив сценариев для сравнения. Если не указано или один — простой тренд

**Пример использования:**
```
get-metrics-monthly:
  project_id: 1
  key: "income"
  months: 6
  scenarios: ["baseline", "optimistic"]
```

### get-metrics-summary

Получить сводку метрик: текущий месяц vs прошлый месяц с процентом изменения.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `keys` (optional, array) - Массив суффиксов ключей метрик (например: ["income", "expense"]). Если не указано — все метрики проекта

**Пример использования:**
```
get-metrics-summary:
  project_id: 1
  keys: ["income", "expense", "balance"]
```

### get-global-metrics

Получить глобальные метрики по всем проектам: income, expense, balance.

**Параметры:** нет

**Пример ответа:**
```json
{
  "summaries": [
    { "key": "g:income", "current": 500000, "prev_month": 450000, "change_pct": 11.1 },
    { "key": "g:expense", "current": 300000, "prev_month": 280000, "change_pct": 7.1 },
    { "key": "g:balance", "current": 200000, "prev_month": 170000, "change_pct": 17.6 }
  ]
}
```

### recompute-metrics

Пересчитать все метрики проекта: материализовать из транзакций и пересчитать формулы. Используй после массовых изменений данных.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `from` (optional, string) - Дата начала (YYYY-MM-DD, default: 12 месяцев назад)
- `to` (optional, string) - Дата окончания (YYYY-MM-DD, default: 6 месяцев вперед)

**Пример использования:**
```
recompute-metrics:
  project_id: 1
  from: "2025-01-01"
  to: "2026-06-30"
```

**Пример ответа:**
```json
{
  "materialized": 156,
  "computed": 12,
  "duration_ms": 234
}
```

---

## 9. Indicators (Показатели)

Индикаторы — вычисляемые показатели на основе формул, ссылающихся на другие метрики проекта. Например, "Чистый доход" = income - expense, или "НДФЛ 13%" = income * 0.13.

### Синтаксис формул

- **Ссылки на метрики**: `p{p}:income`, `p{p}:expense`, `p{p}:balance` (где `{p}` заменяется на ID проекта)
- **Ссылки на другие индикаторы**: `i:p{p}:suffix`
- **Операторы**: `+`, `-`, `*`, `/`
- **Скобки**: `(a + b) * c`
- **Функции**: `max()`, `min()`, `abs()`, `round()`

**Примеры формул:**
- Чистый доход: `p{p}:income - p{p}:expense`
- НДФЛ 13%: `p{p}:income * 0.13`
- Норма сбережений: `(p{p}:income - p{p}:expense) / p{p}:income`

### list-indicators

Получить все индикаторы проекта.

**Параметры:**
- `project_id` (required, integer) - ID проекта

### get-indicator

Получить детали индикатора.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `indicator_id` (required, integer) - ID индикатора

### prepare-indicator

Валидация данных и предпросмотр перед созданием индикатора. **Всегда вызывай перед create-indicator.**

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `suffix` (required, string) - Уникальный суффикс ключа (lowercase, цифры, подчеркивания; начинается с буквы). Пример: "net_income"
- `name` (required, string) - Человекочитаемое название
- `expression` (required, string) - Формула. Используй `{p}` как плейсхолдер ID проекта
- `unit` (optional, string) - Единица измерения: money, percent, number (default: money)
- `display_format` (optional, string) - Формат отображения: currency, percent, number (default: currency)
- `icon` (optional, string) - Идентификатор иконки
- `color` (optional, string) - Цвет в HEX: #FF5733
- `sort_order` (optional, integer) - Порядок сортировки (default: 0)
- `show_scenario_diff` (optional, boolean) - Показывать сравнение сценариев (default: false)

**Пример создания индикатора:**
```
1. prepare-indicator:
   project_id: 1
   suffix: "net_income"
   name: "Чистый доход"
   expression: "p{p}:income - p{p}:expense"
   unit: "money"
   display_format: "currency"

2. [показать preview, получить подтверждение]

3. create-indicator с теми же данными
```

### create-indicator

Создать индикатор (после prepare-indicator и подтверждения пользователя).

Параметры те же что и prepare-indicator.

### update-indicator

Обновить индикатор. Передавай только изменяемые поля.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `indicator_id` (required, integer) - ID индикатора
- `name` (optional, string) - Новое название
- `expression` (optional, string) - Новая формула. Используй `{p}` как плейсхолдер
- `unit` (optional, string) - Единица: money, percent, number
- `display_format` (optional, string) - Формат: currency, percent, number
- `icon` (optional, string) - Иконка
- `color` (optional, string) - Цвет в HEX
- `sort_order` (optional, integer) - Порядок сортировки
- `show_scenario_diff` (optional, boolean) - Показывать сравнение сценариев

### delete-indicator

Удалить индикатор. Нельзя удалить если другие индикаторы зависят от него.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `indicator_id` (required, integer) - ID индикатора

### validate-expression

Проверить формулу без создания индикатора. Полезно для отладки выражений.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `expression` (required, string) - Формула для проверки. Используй `{p}` как плейсхолдер
- `own_key` (optional, string) - Ключ редактируемого индикатора (исключить из проверки циклических зависимостей)

**Пример ответа:**
```json
{
  "valid": true,
  "errors": [],
  "dependencies": ["p:1:income", "p:1:expense"]
}
```

---

## 10. Manual Metrics (Ручные метрики)

Ручные метрики — пользовательские данные, вводимые вручную: количество клиентов, произвольные KPI и т.д. Серия создается автоматически при первой записи значения.

### list-manual-metrics

Получить все ручные метрики проекта.

**Параметры:**
- `project_id` (required, integer) - ID проекта

### store-manual-metric

Записать значение ручной метрики. Серия создается автоматически если не существует (upsert по ключу+периоду).

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `key` (required, string) - Ключ метрики (например: "u:p5:clients")
- `period` (required, string) - Дата периода (YYYY-MM-DD)
- `value` (required, integer) - Значение метрики (целое число)

**Пример использования:**
```
store-manual-metric:
  project_id: 5
  key: "u:p5:clients"
  period: "2026-03-01"
  value: 42
```

**Пример ответа:**
```json
{
  "success": true,
  "message": "Manual metric value stored successfully",
  "metric_value": {
    "key": "u:p5:clients",
    "date": "2026-03-01",
    "value": 42,
    "source": "manual"
  }
}
```

---

## 11. Разработка новых Tools

### Структура файлов

```
app/Mcp/
├── Servers/
│   └── FinTrackServer.php      # Регистрация tools (defaultPaginationLength = 100)
├── Tools/
│   ├── Accounts/               # list, get, prepare, create, update, delete
│   ├── Categories/             # list, get, prepare, create, update, delete
│   ├── Transactions/           # list, get, prepare, create, update, delete, duplicate, transfer
│   ├── Recurrings/             # list, get, prepare, create, update, delete, pause, resume, occurrences
│   ├── Tags/                   # list, get, prepare, create, update, delete
│   ├── Projects/               # list, get, prepare, create, update, delete
│   ├── Metrics/                # get-range, get-monthly, get-summary, get-global, recompute
│   ├── Indicators/             # list, get, prepare, create, update, delete, validate-expression
│   ├── ManualMetrics/          # list, store
│   ├── Dashboard/              # get-dashboard-stats
│   ├── Currencies/             # list-currencies
│   └── Suggestions/            # get-suggestions
└── Validation/
    ├── AccountRules.php        # create() + update() методы
    ├── CategoryRules.php
    ├── TransactionRules.php
    ├── RecurringRules.php
    └── TagRules.php
```

### Ключевые правила

- Используй `$request->get('field')` (не `$request->input()`)
- Проверяй принадлежность сущности пользователю перед update/delete
- Всегда инжектируй сервис, не используй `Model::query()` напрямую
- Используй API Resources для форматирования ответов

### Шаблон нового Tool

```php
<?php

declare(strict_types=1);

namespace App\Mcp\Tools\{Entity};

use App\Http\Resources\{Entity}Resource;
use App\Services\{Entity}Service;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\ResponseFactory;
use Laravel\Mcp\Server\Tool;

class My{Entity}Tool extends Tool
{
    protected string $name = 'my-{entity}';
    protected string $title = 'My {Entity} Tool';
    protected string $description = 'Описание инструмента';

    public function handle(Request $request, {Entity}Service $service): Response|ResponseFactory
    {
        $user = $request->user();

        if (! $user) {
            return Response::text('Authentication required');
        }

        $validated = $request->validate([
            'param' => 'required|string',
        ]);

        $result = $service->doSomething($user, $validated);

        $data = (new {Entity}Resource($result))->toArray(request());

        return Response::structured([
            'success' => true,
            'data' => $data,
        ]);
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'param' => $schema->string()
                ->description('Описание параметра')
                ->required(),
        ];
    }

    public function outputSchema(JsonSchema $schema): array
    {
        return [
            'success' => $schema->boolean()->required(),
            'data' => $schema->object()->required(),
        ];
    }
}
```

### Регистрация Tool

Добавь класс в `FinTrackServer::$tools`:

```php
protected array $tools = [
    // ... existing tools
    \App\Mcp\Tools\{Entity}\My{Entity}Tool::class,
];
```

### Ограничение пагинации

По умолчанию Laravel MCP показывает только 15 tools.
В `FinTrackServer` увеличено до 100 (покрывает все 58 tools):

```php
public int $defaultPaginationLength = 100;
```

---

## Отладка

### Проверка количества доступных tools

```bash
# Продакшн
MCP_URL="https://api.equity.su/mcp"
# Локальная
# MCP_URL="http://fintrack.test/mcp"

curl -s -X POST $MCP_URL \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \
  | jq '.result.tools | length'
# Ожидаем: 58
```

### Тестовый вызов

```bash
curl -s -X POST https://api.equity.su/mcp \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"tools/call",
    "params":{
      "name":"list-transactions",
      "arguments":{"from_date":"2026-02-01","to_date":"2026-02-28","limit":10}
    }
  }' | jq '.result.content[0].text' | jq .
```

### Логи

```bash
tail -f storage/logs/laravel.log | grep -i mcp
```
