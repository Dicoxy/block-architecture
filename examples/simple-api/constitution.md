# Simple API Constitution

> Правила для минимального примера.
> Упрощённая версия для тестирования.

## 1. Технологический стек

- Python: 3.11+
- Framework: FastAPI
- Server: uvicorn

## 2. Стандарты кода

### Стиль
- Форматирование: не критично для примера
- Type hints: желательны

### Структура
```
src/
├── main.py         # Точка входа
└── routes/
    └── hello.py    # Endpoints
```

## 3. API Design

- Формат ответа: JSON
- Content-Type: application/json

## 4. Что запрещено

- Внешние зависимости кроме FastAPI и uvicorn
- Сложная логика
- База данных

## 5. Чеклист

- [ ] API отвечает на GET /
- [ ] API отвечает на GET /time
- [ ] Ответы в формате JSON
