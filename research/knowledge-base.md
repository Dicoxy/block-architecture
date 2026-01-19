# База знаний агента-архитектора

Что агент должен знать и уметь находить.

## Категории знаний

### 1. Паттерны архитектуры

- Микросервисы vs монолит
- Event-driven architecture
- Serverless patterns
- API-first design

### 2. Типовые блоки

| Блок | Типичный вход | Типичный выход |
|------|---------------|----------------|
| Auth | credentials | token |
| API Gateway | request + token | routed request |
| Database | query | data |
| Cache | key | value |
| Queue | message | confirmation |
| Notification | event | delivery status |

### 3. Интеграции

- Telegram Bot API
- OpenAI / Anthropic APIs
- Payment systems
- Social media APIs
- Cloud services (AWS, GCP, etc)

### 4. Тенденции 2026

(Обновлять регулярно через web search)

- AI coding agents
- MCP protocol
- Long-horizon agents
- Vibe coding practices

## Как агент использует базу

1. Получает идею
2. Ищет релевантные паттерны
3. Ищет типовые блоки под задачу
4. Проверяет актуальные тенденции (web search)
5. Составляет план на основе найденного

## Обновление базы

База должна обновляться:
- Автоматически: web search при каждом новом проекте
- Вручную: добавление новых паттернов после успешных проектов
