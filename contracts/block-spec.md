# Block Contract Specification v0.3

Контракт блока — описание того, что блок принимает и что отдаёт.

## Философия

Контракт — это **соглашение**:
- Агент уровня знает ТОЛЬКО свой контракт
- Он не знает что внутри других блоков
- Если результат соответствует контракту — блок успешен

Контракт отвечает на вопросы:
1. Что я получу на вход?
2. Что я должен выдать?
3. Как проверить что я справился?

---

## Структура контракта

```yaml
# contracts/auth.yaml

version: "0.3"

# ============================================
# СЕКЦИЯ 1: ИДЕНТИФИКАЦИЯ
# ============================================
block:
  id: "auth"
  name: "Сервис аутентификации"
  description: |
    Обеспечивает JWT аутентификацию пользователей.
    Создаёт токены, валидирует их, управляет сессиями.

# ============================================
# СЕКЦИЯ 2: ВХОДНЫЕ ДАННЫЕ
# ============================================
input:
  # Входы от других блоков
  from_blocks:
    - source_block: "config"
      source_output: "jwt_secret"
      local_name: "secret_key"
      required: true
      
    - source_block: "database"
      source_output: "connection_pool"
      local_name: "db_pool"
      required: true
  
  # Входы из constitution/environment
  from_environment:
    - name: "token_expiry_hours"
      type: "integer"
      default: 24
      
    - name: "algorithm"
      type: "string"
      default: "HS256"
      allowed_values: ["HS256", "RS256"]

# ============================================
# СЕКЦИЯ 3: ВЫХОДНЫЕ ДАННЫЕ
# ============================================
output:
  # Основные выходы которые используют другие блоки
  exports:
    - name: "token_validator"
      type: "function"
      description: "Функция для валидации JWT токена"
      signature: |
        async def validate_token(token: str) -> UserPayload | None
      
    - name: "middleware"
      type: "object"
      description: "FastAPI middleware для защиты роутов"
      interface: |
        class AuthMiddleware:
            async def __call__(request: Request) -> Response
      
    - name: "create_token"
      type: "function"
      description: "Функция создания нового токена"
      signature: |
        def create_token(user_id: str, extra_claims: dict = {}) -> str
  
  # Артефакты (файлы, которые создаёт блок)
  artifacts:
    - path: "src/auth/service.py"
      description: "Основной сервис аутентификации"
      
    - path: "src/auth/middleware.py"
      description: "Middleware для FastAPI"
      
    - path: "src/auth/models.py"
      description: "Pydantic модели"
      
    - path: "tests/test_auth.py"
      description: "Тесты"

# ============================================
# СЕКЦИЯ 4: КРИТЕРИИ УСПЕХА
# ============================================
success_criteria:
  # Обязательные критерии
  required:
    - id: "files_created"
      description: "Все артефакты созданы"
      check: "all artifacts exist"
      
    - id: "exports_available"
      description: "Все exports доступны для импорта"
      check: "imports work without errors"
      
    - id: "tests_pass"
      description: "Тесты проходят"
      check: "pytest returns 0"
  
  # Желательные критерии
  optional:
    - id: "coverage"
      description: "Покрытие тестами > 80%"
      check: "coverage report > 80%"

# ============================================
# СЕКЦИЯ 5: ОГРАНИЧЕНИЯ
# ============================================
constraints:
  # Технические ограничения
  technical:
    - "Использовать python-jose для JWT"
    - "Async операции для всех DB запросов"
    - "Pydantic v2 для моделей"
  
  # Ограничения из constitution (ссылки)
  from_constitution:
    - "security.no_secrets_in_code"
    - "style.type_hints_required"
    - "testing.minimum_coverage"
  
  # Что НЕ делать
  forbidden:
    - "Хранить пароли в plain text"
    - "Использовать MD5 для хеширования"
    - "Делать синхронные вызовы к БД"

# ============================================
# СЕКЦИЯ 6: ПРИМЕРЫ
# ============================================
examples:
  # Пример использования exports
  usage: |
    from src.auth.service import create_token, validate_token
    from src.auth.middleware import AuthMiddleware
    
    # Создание токена
    token = create_token(user_id="123", extra_claims={"role": "admin"})
    
    # Валидация
    payload = await validate_token(token)
    if payload:
        print(f"User: {payload.user_id}")
    
    # Middleware
    app.add_middleware(AuthMiddleware)
  
  # Пример ожидаемого результата
  expected_result: |
    {
      "token_validator": "<function>",
      "middleware": "<AuthMiddleware class>",
      "create_token": "<function>",
      "artifacts": [
        "src/auth/service.py",
        "src/auth/middleware.py",
        "src/auth/models.py",
        "tests/test_auth.py"
      ]
    }

# ============================================
# СЕКЦИЯ 7: КОНФИГУРАЦИЯ ВЫПОЛНЕНИЯ
# ============================================
execution:
  # Сколько попыток при ошибке
  max_retries: 3
  
  # Таймаут в минутах
  timeout_minutes: 15
  
  # Что делать при неудаче
  on_failure: "escalate"  # retry | escalate | skip
  
  # Приоритет (для параллельного выполнения)
  priority: "high"  # low | medium | high | critical
  
  # Требуемые инструменты
  tools:
    - "file_write"
    - "file_read"
    - "shell_execute"
    - "python_run"
```

---

## Типы данных в контрактах

### Примитивы

| Тип | Описание | Пример |
|-----|----------|--------|
| string | Текст | "hello" |
| integer | Целое число | 42 |
| float | Дробное число | 3.14 |
| boolean | true/false | true |
| null | Пустое значение | null |

### Составные типы

| Тип | Описание | Пример |
|-----|----------|--------|
| object | Структура с полями | {name: string, age: integer} |
| array | Список | [string] |
| function | Callable | async def f(x: int) -> str |
| class | Класс/объект | class MyClass |
| file | Путь к файлу | "src/main.py" |

### Специальные типы

| Тип | Описание |
|-----|----------|
| any | Любой тип (избегать!) |
| union | Один из типов: string \| null |
| optional | Может отсутствовать |

---

## Связь между блоками через контракты

```
┌─────────────────────────────────────────────────────────────┐
│ Блок A (config)                                             │
│                                                             │
│ output.exports:                                             │
│   - name: "jwt_secret"      ←────────────────────┐         │
│     type: "string"                               │         │
└─────────────────────────────────────────────────────────────┘
                                                   │
                                                   │ связь через
                                                   │ source_output
                                                   │
┌─────────────────────────────────────────────────────────────┐
│ Блок B (auth)                                               │
│                                                             │
│ input.from_blocks:                                          │
│   - source_block: "config"                                  │
│     source_output: "jwt_secret"  ←───────────────┘         │
│     local_name: "secret_key"                                │
└─────────────────────────────────────────────────────────────┘
```

Оркестратор проверяет что:
1. Блок A экспортирует "jwt_secret"
2. Тип совпадает
3. Блок A завершён успешно до запуска блока B

---

## Валидация контракта

### При загрузке (статическая)

- [ ] YAML синтаксически корректен
- [ ] Все обязательные поля присутствуют
- [ ] Типы указаны корректно
- [ ] Ссылки на constitution валидны

### При связывании (граф)

- [ ] Все source_block существуют в манифесте
- [ ] Все source_output существуют в контрактах источников
- [ ] Типы входов и выходов совместимы
- [ ] Нет циклических зависимостей

### При выполнении (runtime)

- [ ] Все required входы получены
- [ ] Результат соответствует output schema
- [ ] Все success_criteria.required выполнены
- [ ] Артефакты созданы

---

## Минимальный контракт

Для простого блока:

```yaml
version: "0.3"

block:
  id: "hello"
  name: "Hello Endpoint"
  description: "Создаёт endpoint GET /"

input:
  from_blocks: []
  from_environment: []

output:
  exports:
    - name: "router"
      type: "object"
      description: "FastAPI router"
  artifacts:
    - path: "src/routes/hello.py"

success_criteria:
  required:
    - id: "file_exists"
      description: "Файл создан"
      check: "artifact exists"

execution:
  max_retries: 2
  timeout_minutes: 5
  on_failure: "retry"
```

---

## Anti-patterns

### ❌ Слишком широкий контракт

```yaml
output:
  exports:
    - name: "everything"
      type: "any"  # Плохо! Нет контракта
```

### ❌ Зависимость от внутренностей другого блока

```yaml
input:
  from_blocks:
    - source_block: "auth"
      source_output: "_internal_cache"  # Плохо! Это internal
```

### ❌ Отсутствие критериев успеха

```yaml
success_criteria:
  required: []  # Плохо! Как понять что блок работает?
```

### ✅ Хороший контракт

- Чёткие типы
- Понятные описания
- Конкретные критерии успеха
- Примеры использования
