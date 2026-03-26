# Equity MCP Server - Полное Руководство

**Version 1.3.0**
Equity
March 2026

> **Примечание:**
> Этот документ предназначен для AI агентов и LLM при работе с MCP сервером Equity.
> Содержит детальные примеры использования всех 70 инструментов.

---

## Содержание

1. [Обзор](#1-обзор)
2. [Accounts (Счета)](#2-accounts-счета)
3. [Categories (Категории)](#3-categories-категории)
4. [Transactions (Транзакции)](#4-transactions-транзакции)
5. [Recurring Payments (Повторяющиеся платежи)](#5-recurring-payments-повторяющиеся-платежи)
6. [Occurrences (Плановые платежи)](#6-occurrences-плановые-платежи)
7. [Occurrence Overrides (Пропуск/перенос)](#7-occurrence-overrides-пропускперенос)
8. [Links (Привязка транзакций)](#8-links-привязка-транзакций)
9. [Tags (Теги)](#9-tags-теги)
10. [Projects (Проекты)](#10-projects-проекты)
11. [Dashboard (Статистика)](#11-dashboard-статистика)
12. [Pivot (Аналитика)](#12-pivot-аналитика)
13. [Metrics (Метрики)](#13-metrics-метрики)
14. [Indicators (Показатели)](#14-indicators-показатели)
15. [Manual Metrics (Ручные метрики)](#15-manual-metrics-ручные-метрики)
16. [Currencies & Suggestions](#16-currencies--suggestions)
17. [Разработка новых Tools](#17-разработка-новых-tools)

---

## 1. Обзор

### Конфигурация

| Среда | URL |
|-------|-----|
| Продакшн | `https://api.equity.su/mcp` |

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

### Формат данных

- **Даты**: ISO 8601 (YYYY-MM-DD)
- **Суммы в occurrences/pivot**: в копейках/центах (integer). 150000 = 1500.00 RUB
- **Суммы в остальных tools**: в основной валюте (рублях). НЕ в копейках
- **Валюты**: ISO 4217 (RUB, USD, EUR)

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

### duplicate-transaction

Дублировать существующую транзакцию.

**Параметры:**
- `transaction_id` (required, integer) - ID транзакции для дублирования

### transfer-funds

Перевод между счетами. Поддерживает кросс-валютные переводы с автоматическим расчётом курса.

**Параметры:**
- `from_account_id` (required, integer) - ID счета-источника
- `to_account_id` (required, integer) - ID счета-получателя
- `amount` (required, number) - Сумма перевода
- `description` (optional, string) - Описание
- `transaction_date` (optional, string) - Дата YYYY-MM-DD

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

### pause-recurring

Приостановить повторяющийся платеж.

**Параметры:**
- `recurring_id` (required, integer) - ID платежа

### resume-recurring

Возобновить приостановленный платеж.

**Параметры:**
- `recurring_id` (required, integer) - ID платежа

### get-recurring-occurrences

Получить запланированные вхождения конкретного повторяющегося платежа.

**Параметры:**
- `recurring_id` (required, integer) - ID платежа
- `from` (optional, string) - Дата начала (YYYY-MM-DD)
- `to` (optional, string) - Дата окончания (YYYY-MM-DD)

---

## 6. Occurrences (Плановые платежи)

Occurrences — вхождения повторяющихся платежей за период. Показывают статус оплаты, суммы, категории и счета. Используются для план/факт анализа.

### list-occurrences

Все вхождения за период со статусами оплаты.

**Параметры:**
- `from` (required, string) - Дата начала (YYYY-MM-DD)
- `to` (required, string) - Дата окончания (YYYY-MM-DD)
- `recurring_id` (optional, integer) - Фильтр по ID повторяющегося платежа
- `type` (optional, string) - Фильтр по типу: expense, income, transfer
- `is_paid` (optional, boolean) - Фильтр: true = оплачено, false = не оплачено
- `category_id` (optional, integer) - Фильтр по категории
- `account_id` (optional, integer) - Фильтр по счету

**Пример ответа:**
```json
{
  "data": [...],
  "total": 15,
  "period": { "from": "2026-03-01", "to": "2026-03-31" }
}
```

**Пример использования:**
```
User: Какие платежи запланированы на март?
AI: [вызывает list-occurrences с from: "2026-03-01", to: "2026-03-31"]
    Показывает список с разделением на оплаченные/неоплаченные
```

### get-occurrences-totals

Агрегация: запланировано/оплачено/не оплачено по типам.

**Параметры:**
- `from` (required, string) - Дата начала (YYYY-MM-DD)
- `to` (required, string) - Дата окончания (YYYY-MM-DD)
- `type` (optional, string) - Фильтр: expense или income
- `project_id` (optional, integer) - Фильтр по проекту

**Пример ответа:**
```json
{
  "planned_income": 500000,
  "planned_expense": 350000,
  "paid_income": 500000,
  "paid_expense": 200000,
  "unpaid_income": 0,
  "unpaid_expense": 150000,
  "currency": "RUB"
}
```

**Пример использования:**
```
User: Сколько я должен заплатить в этом месяце?
AI: [вызывает get-occurrences-totals с type: "expense"]
    Запланировано расходов: 3500 ₽
    Оплачено: 2000 ₽
    Осталось оплатить: 1500 ₽
```

### get-occurrences-by-category

Группировка вхождений по категориям.

**Параметры:**
- `from` (required, string) - Дата начала (YYYY-MM-DD)
- `to` (required, string) - Дата окончания (YYYY-MM-DD)
- `type` (optional, string) - Фильтр: expense или income

**Пример ответа:**
```json
{
  "data": [
    {
      "category_id": 5,
      "category_name": "Подписки",
      "type": "expense",
      "total_planned": 150000,
      "total_paid": 100000,
      "total_unpaid": 50000,
      "occurrences_count": 3
    }
  ],
  "period": { "from": "2026-03-01", "to": "2026-03-31" }
}
```

### get-upcoming-occurrences

Ближайшие неоплаченные вхождения. Просроченные (overdue) показываются первыми.

**Параметры:**
- `days` (optional, integer) - Дней вперед (default: 7, max: 365)
- `limit` (optional, integer) - Максимум результатов (default: 20, max: 100)

**Пример ответа:**
```json
{
  "data": [
    {
      "recurring_id": 3,
      "recurring_name": "Аренда",
      "occurrence_date": "2026-03-15",
      "amount": 3500000,
      "type": "expense",
      "category": { "id": 2, "name": "Жилье" },
      "account": { "id": 1, "name": "Сбербанк" },
      "is_paid": false,
      "is_overdue": true,
      "has_override": false,
      "priority": "high"
    }
  ],
  "total": 5
}
```

**Пример использования:**
```
User: Что нужно оплатить на этой неделе?
AI: [вызывает get-upcoming-occurrences с days: 7]
    ⚠️ Просрочено:
    - Аренда (15 марта) — 35 000 ₽

    Предстоит:
    - Netflix (20 марта) — 799 ₽
    - Интернет (22 марта) — 500 ₽
```

---

## 7. Occurrence Overrides (Пропуск/перенос)

Overrides позволяют пропустить или перенести конкретное вхождение повторяющегося платежа без изменения самого расписания.

### list-occurrence-overrides

Все overrides для конкретного повторяющегося платежа.

**Параметры:**
- `recurring_id` (required, integer) - ID повторяющегося платежа

**Пример ответа:**
```json
{
  "data": [
    {
      "id": 1,
      "recurring_id": 3,
      "occurrence_date": "2026-04-01",
      "type": "skip",
      "new_date": null,
      "overridden_amount": null,
      "reason": "Отпуск",
      "created_at": "2026-03-20T10:00:00Z"
    }
  ]
}
```

### create-occurrence-override

Пропустить или перенести вхождение.

**Параметры:**
- `recurring_id` (required, integer) - ID повторяющегося платежа
- `occurrence_date` (required, string) - Дата вхождения (YYYY-MM-DD)
- `type` (required, string) - Тип: `skip` (пропустить) или `reschedule` (перенести)
- `new_date` (optional, string) - Новая дата (YYYY-MM-DD, обязательно для reschedule)
- `reason` (optional, string) - Причина (max 500 символов)

**Пример использования:**
```
User: Пропусти оплату аренды 1 апреля, я в отпуске
AI: [вызывает create-occurrence-override]
    recurring_id: 3
    occurrence_date: "2026-04-01"
    type: "skip"
    reason: "Отпуск"
    ✓ Вхождение 1 апреля пропущено.

User: Перенеси оплату интернета с 15 на 20 марта
AI: [вызывает create-occurrence-override]
    recurring_id: 5
    occurrence_date: "2026-03-15"
    type: "reschedule"
    new_date: "2026-03-20"
    ✓ Оплата перенесена на 20 марта.
```

### delete-occurrence-override

Отменить пропуск/перенос — вернуть вхождение в исходное расписание.

**Параметры:**
- `recurring_id` (required, integer) - ID повторяющегося платежа
- `occurrence_date` (required, string) - Дата вхождения (YYYY-MM-DD)

---

## 8. Links (Привязка транзакций)

Links связывают реальные транзакции с повторяющимися платежами. Когда транзакция привязана — вхождение считается оплаченным.

### link-transaction-to-recurring

Привязать транзакцию к повторяющемуся платежу (отметить как оплаченный).

**Параметры:**
- `recurring_id` (required, integer) - ID повторяющегося платежа
- `transaction_id` (required, integer) - ID транзакции

**Пример ответа:**
```json
{
  "message": "Transaction linked successfully",
  "recurring_id": 3,
  "transaction_id": 42
}
```

**Пример использования:**
```
User: Я оплатил аренду, транзакция 42 — это оплата по recurring 3
AI: [вызывает link-transaction-to-recurring]
    recurring_id: 3
    transaction_id: 42
    ✓ Транзакция привязана. Вхождение отмечено как оплаченное.
```

### unlink-transaction-from-recurring

Отвязать транзакцию. Вхождение снова станет "не оплачено".

**Параметры:**
- `recurring_id` (required, integer) - ID повторяющегося платежа
- `transaction_id` (required, integer) - ID транзакции

---

## 9. Tags (Теги)

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

## 10. Projects (Проекты)

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

## 11. Dashboard (Статистика)

### get-dashboard-stats

Общая статистика: доходы, расходы, баланс за период.

**Параметры:** нет (использует текущий месяц)

---

## 12. Pivot (Аналитика)

Сводные таблицы (pivot tables) — аналитика по категориям за период. Показывают разбивку доходов/расходов по категориям и временным периодам.

### get-dashboard-pivot

Главная сводная таблица бюджета. Разбивка по категориям за период.

**Параметры:**
- `period` (required, string) - Период: week, month, quarter, year, custom
- `from` (optional, string) - Дата начала (YYYY-MM-DD, обязательно для custom)
- `to` (optional, string) - Дата окончания (YYYY-MM-DD, обязательно для custom)
- `type` (optional, string) - Фильтр: expense или income
- `account_id` (optional, integer) - Фильтр по счету

**Пример использования:**
```
User: Покажи бюджет за март по категориям
AI: [вызывает get-dashboard-pivot]
    period: "month"
    type: "expense"

    Расходы за март:
    - Жилье: 35 000 ₽
    - Продукты: 15 000 ₽
    - Транспорт: 5 000 ₽
    Итого: 55 000 ₽
```

### get-project-pivot

Сводная таблица по конкретному проекту и его подпроектам.

**Параметры:**
- `project_id` (required, integer) - ID проекта
- `period` (required, string) - Период: week, month, quarter, year, custom
- `from` (optional, string) - Дата начала (YYYY-MM-DD, обязательно для custom)
- `to` (optional, string) - Дата окончания (YYYY-MM-DD, обязательно для custom)

### get-subproject-pivot

Разбивка по подпроектам внутри проекта.

**Параметры:**
- `project_id` (required, integer) - ID родительского проекта
- `from` (optional, string) - Дата начала (YYYY-MM-DD, default: начало текущего месяца)
- `to` (optional, string) - Дата окончания (YYYY-MM-DD, default: конец текущего месяца)
- `base_currency` (optional, string) - Валюта конвертации (RUB, USD, EUR)

**Пример ответа:**
```json
{
  "subprojects": [
    {
      "project_id": 2,
      "project_name": "Маркетинг",
      "income": 0,
      "expense": 150000,
      "balance": -150000
    },
    {
      "project_id": 3,
      "project_name": "Разработка",
      "income": 0,
      "expense": 300000,
      "balance": -300000
    }
  ]
}
```

---

## 13. Metrics (Метрики)

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

**Пример ответа:**
```json
{
  "materialized": 156,
  "computed": 12,
  "duration_ms": 234
}
```

---

## 14. Indicators (Показатели)

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

## 15. Manual Metrics (Ручные метрики)

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

## 16. Currencies & Suggestions

### list-currencies

Получить список доступных валют для создания счетов.

**Параметры:** нет

### get-suggestions

Автозаполнение описаний, категорий, тегов и счетов на основе истории транзакций.

**Параметры:**
- `query` (required, string) - Поисковый запрос

---

## 17. Разработка новых Tools

### Структура файлов

```
app/Mcp/
├── Servers/
│   └── EquityServer.php           # Регистрация tools (defaultPaginationLength = 100)
├── Tools/
│   ├── Accounts/                  # list, get, prepare, create, update, delete
│   ├── Categories/                # list, get, prepare, create, update, delete
│   ├── Transactions/              # list, get, prepare, create, update, delete, duplicate, transfer
│   ├── Recurrings/                # list, get, prepare, create, update, delete, pause, resume, occurrences
│   ├── Occurrences/               # list, totals, by-category, upcoming
│   ├── Overrides/                 # list, create, delete
│   ├── Links/                     # link, unlink
│   ├── Tags/                      # list, get, prepare, create, update, delete
│   ├── Projects/                  # list, get, prepare, create, update, delete
│   ├── Metrics/                   # get-range, get-monthly, get-summary, get-global, recompute
│   ├── Indicators/                # list, get, prepare, create, update, delete, validate-expression
│   ├── ManualMetrics/             # list, store
│   ├── Dashboard/                 # get-dashboard-stats
│   ├── Pivot/                     # dashboard, project, subproject
│   ├── Currencies/                # list-currencies
│   └── Suggestions/               # get-suggestions
└── Validation/
    ├── AccountRules.php           # create() + update() методы
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

Добавь класс в `EquityServer::$tools`:

```php
protected array $tools = [
    // ... existing tools
    \App\Mcp\Tools\{Entity}\My{Entity}Tool::class,
];
```

### Ограничение пагинации

По умолчанию Laravel MCP показывает только 15 tools.
В `EquityServer` увеличено до 100 (покрывает все 70 tools):

```php
public int $defaultPaginationLength = 100;
```

---

## Отладка

### Проверка количества доступных tools

```bash
MCP_URL="https://api.equity.su/mcp"

curl -s -X POST $MCP_URL \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \
  | jq '.result.tools | length'
# Ожидаем: 70
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
