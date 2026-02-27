# FinTrack MCP Server - Полное Руководство

**Version 1.1.0**
FinTrack
February 2026

> **Примечание:**
> Этот документ предназначен для AI агентов и LLM при работе с MCP сервером FinTrack.
> Содержит детальные примеры использования всех 30 инструментов.

---

## Содержание

1. [Обзор](#1-обзор)
2. [Accounts (Счета)](#2-accounts-счета)
3. [Categories (Категории)](#3-categories-категории)
4. [Transactions (Транзакции)](#4-transactions-транзакции)
5. [Recurring Payments (Повторяющиеся платежи)](#5-recurring-payments-повторяющиеся-платежи)
6. [Tags (Теги)](#6-tags-теги)
7. [Разработка новых Tools](#7-разработка-новых-tools)

---

## 1. Обзор

### Конфигурация

MCP сервер доступен по адресу `http://fintrack.test/mcp` с авторизацией через Sanctum Bearer Token.

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

## 7. Разработка новых Tools

### Структура файлов

```
app/Mcp/
├── Servers/
│   └── FinTrackServer.php      # Регистрация tools (defaultPaginationLength = 50)
├── Tools/
│   ├── Accounts/               # list, get, prepare, create, update, delete
│   ├── Categories/             # list, get, prepare, create, update, delete
│   ├── Transactions/           # list, get, prepare, create, update, delete
│   ├── Recurrings/             # list, get, prepare, create, update, delete
│   └── Tags/                   # list, get, prepare, create, update, delete
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
В `FinTrackServer` увеличено до 50 (покрывает все 30 tools):

```php
public int $defaultPaginationLength = 50;
```

---

## Отладка

### Проверка количества доступных tools

```bash
curl -s -X POST http://fintrack.test/mcp \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \
  | jq '.result.tools | length'
# Ожидаем: 30
```

### Тестовый вызов

```bash
curl -s -X POST http://fintrack.test/mcp \
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
