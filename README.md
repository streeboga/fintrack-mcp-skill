# Equity.ru MCP Skill

AI-скилл для управления личными финансами через [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).
Предоставляет **58 инструментов** для работы с финансовыми данными в приложении [Equity](https://equity.su).

---

## Структура

| Файл | Описание |
|------|----------|
| `SKILL.md` | Краткая справка: сущности, паттерны, типы данных |
| `AGENTS.md` | Полное руководство: все 58 инструментов с параметрами и примерами |

## Сущности и инструменты

| Сущность | Описание | Кол-во |
|----------|----------|--------|
| Accounts | Банковские счета, карты, наличные | 6 |
| Categories | Категории доходов и расходов | 6 |
| Transactions | Финансовые операции, дубликаты, переводы | 8 |
| Recurrings | Повторяющиеся платежи (пауза, возобновление, вхождения) | 9 |
| Tags | Метки для организации | 6 |
| Projects | Проекты и подпроекты | 6 |
| Metrics | Автоматические метрики проекта | 5 |
| Indicators | Вычисляемые показатели по формулам | 7 |
| Manual Metrics | Ручные метрики пользователя | 2 |
| Dashboard | Статистика и обзор | 1 |
| Currencies | Доступные валюты | 1 |
| Suggestions | Автозаполнение | 1 |

## Подключение

| Среда | URL |
|-------|-----|
| Продакшн | `https://api.equity.su/mcp` |
| Локальная | `http://fintrack.test/mcp` |

Авторизация: Sanctum Bearer Token.

## Использование

### Cursor

Скопируйте содержимое в `.cursor/skills/equity-mcp/`:

```bash
git clone git@github.com:streeboga/equity.ru-mcp-skill.git .cursor/skills/equity-mcp
```

### Claude Code

Добавьте как зависимость или скопируйте `SKILL.md` и `AGENTS.md` в директорию проекта.

## Лицензия

MIT

---

# Equity.ru MCP Skill (English)

AI skill for personal finance management via [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).
Provides **58 tools** for working with financial data in the [Equity](https://equity.su) app.

## Structure

| File | Description |
|------|-------------|
| `SKILL.md` | Quick reference: entities, patterns, data types |
| `AGENTS.md` | Full guide: all 58 tools with parameters and examples |

## Entities and Tools

| Entity | Description | Count |
|--------|-------------|-------|
| Accounts | Bank accounts, cards, cash | 6 |
| Categories | Income and expense categories | 6 |
| Transactions | Financial operations, duplicates, transfers | 8 |
| Recurrings | Recurring payments (pause, resume, occurrences) | 9 |
| Tags | Labels for organization | 6 |
| Projects | Projects and subprojects | 6 |
| Metrics | Automatic project metrics | 5 |
| Indicators | Computed indicators via formulas | 7 |
| Manual Metrics | User-defined manual metrics | 2 |
| Dashboard | Statistics and overview | 1 |
| Currencies | Available currencies | 1 |
| Suggestions | Autocomplete | 1 |

## Connection

| Environment | URL |
|-------------|-----|
| Production | `https://api.equity.su/mcp` |
| Local | `http://fintrack.test/mcp` |

Authorization: Sanctum Bearer Token.

## Usage

### Cursor

Copy contents into `.cursor/skills/equity-mcp/`:

```bash
git clone git@github.com:streeboga/equity.ru-mcp-skill.git .cursor/skills/equity-mcp
```

### Claude Code

Add as a dependency or copy `SKILL.md` and `AGENTS.md` into your project directory.

## License

MIT
