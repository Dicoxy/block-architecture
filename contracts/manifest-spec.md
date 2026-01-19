# Manifest Specification v0.3

Манифест — центральный артефакт системы. Описывает проект, его блоки, связи и состояние.

## Философия

Манифест — это НЕ код. Это **декларация намерений**:
- Что мы строим
- Из каких частей
- Как части связаны
- Что считается успехом

Агент-архитектор создаёт манифест. Оркестратор исполняет его. Человек валидирует.

---

## Структура манифеста

```yaml
# manifest.yaml

version: "0.3"

# ============================================
# СЕКЦИЯ 1: МЕТАДАННЫЕ ПРОЕКТА
# ============================================
project:
  id: "unique-project-id"
  name: "Human-readable название"
  description: |
    Краткое описание что это за проект
    и какую проблему решает.
  created_at: "2026-01-19T12:00:00Z"
  updated_at: "2026-01-19T12:00:00Z"

# ============================================
# СЕКЦИЯ 2: ИСХОДНАЯ ИДЕЯ
# ============================================
idea:
  raw: |
    Оригинальный текст идеи от человека.
    Без обработки, как есть.
  interpreted: |
    Интерпретация агента-архитектора.
    Что он понял из идеи.
  validated: true  # Человек подтвердил интерпретацию

# ============================================
# СЕКЦИЯ 3: ИССЛЕДОВАНИЕ
# ============================================
research:
  status: "completed"  # pending | in_progress | completed
  
  market:
    - finding: "Существуют аналоги X, Y, Z"
      relevance: "high"
      source: "https://..."
    - finding: "Тренд на использование технологии T"
      relevance: "medium"
      source: "https://..."
  
  technical:
    - finding: "Лучший стек для задачи: Python + FastAPI"
      rationale: "Скорость разработки, async из коробки"
    - finding: "Для очередей использовать Redis"
      rationale: "Простота, скорость, широкая поддержка"
  
  risks:
    - risk: "API провайдера может измениться"
      severity: "medium"
      mitigation: "Абстрагировать через адаптер"

# ============================================
# СЕКЦИЯ 4: CONSTITUTION (ссылка)
# ============================================
constitution:
  path: "./constitution.md"
  checksum: "sha256:abc123..."  # Для проверки что не изменился

# ============================================
# СЕКЦИЯ 5: АРХИТЕКТУРА
# ============================================
architecture:
  total_levels: 3
  total_blocks: 7
  
  # Определение блоков
  blocks:
    # --- Уровень 1: Базовые сервисы ---
    - id: "auth"
      name: "Аутентификация"
      level: 1
      contract: "./contracts/auth.yaml"
      depends_on: []
      
    - id: "database"
      name: "Подключение к БД"
      level: 1
      contract: "./contracts/database.yaml"
      depends_on: []
      
    - id: "config"
      name: "Конфигурация"
      level: 1
      contract: "./contracts/config.yaml"
      depends_on: []
    
    # --- Уровень 2: Бизнес-логика ---
    - id: "user_service"
      name: "Сервис пользователей"
      level: 2
      contract: "./contracts/user_service.yaml"
      depends_on:
        - block: "auth"
          required_output: "token_validator"
        - block: "database"
          required_output: "connection_pool"
          
    - id: "api_gateway"
      name: "API Gateway"
      level: 2
      contract: "./contracts/api_gateway.yaml"
      depends_on:
        - block: "auth"
          required_output: "middleware"
        - block: "config"
          required_output: "routes_config"
    
    # --- Уровень 3: Интеграции ---
    - id: "external_api"
      name: "Внешние API"
      level: 3
      contract: "./contracts/external_api.yaml"
      depends_on:
        - block: "api_gateway"
          required_output: "http_client"
        - block: "config"
          required_output: "api_keys"
          
    - id: "notifications"
      name: "Уведомления"
      level: 3
      contract: "./contracts/notifications.yaml"
      depends_on:
        - block: "user_service"
          required_output: "user_preferences"

# ============================================
# СЕКЦИЯ 6: FLOW (визуализация)
# ============================================
flow:
  diagram: |
    Level 1:  [auth]     [database]     [config]
                 ↓           ↓             ↓
    Level 2:  [user_service]    [api_gateway]
                      ↓              ↓
    Level 3:    [notifications]  [external_api]
  
  parallel_groups:
    - level: 1
      blocks: ["auth", "database", "config"]
      note: "Все три запускаются параллельно"
    - level: 2
      blocks: ["user_service", "api_gateway"]
      note: "Параллельно после завершения уровня 1"
    - level: 3
      blocks: ["notifications", "external_api"]
      note: "Параллельно после своих зависимостей"

# ============================================
# СЕКЦИЯ 7: VALIDATION GATES
# ============================================
validation:
  plan:
    status: "approved"  # draft | pending | approved | rejected
    approved_by: "human"
    approved_at: "2026-01-19T12:30:00Z"
    notes: "Убрали блок кеширования, добавим позже"
  
  gates:
    - after_level: 1
      type: "automatic"  # automatic | human
      criteria: "Все блоки уровня 1 зелёные"
      
    - after_level: 2
      type: "automatic"
      criteria: "API Gateway отвечает на health check"
      
    - after_level: 3
      type: "human"
      criteria: "Человек проверяет интеграции"

# ============================================
# СЕКЦИЯ 8: RUNTIME STATE
# ============================================
runtime:
  status: "running"  # idle | running | paused | completed | failed
  started_at: "2026-01-19T13:00:00Z"
  current_level: 2
  
  blocks:
    auth:
      status: "green"
      started_at: "2026-01-19T13:00:00Z"
      completed_at: "2026-01-19T13:02:15Z"
      result_ref: "./results/auth.json"
      
    database:
      status: "green"
      started_at: "2026-01-19T13:00:00Z"
      completed_at: "2026-01-19T13:01:45Z"
      result_ref: "./results/database.json"
      
    config:
      status: "green"
      started_at: "2026-01-19T13:00:00Z"
      completed_at: "2026-01-19T13:00:30Z"
      result_ref: "./results/config.json"
      
    user_service:
      status: "running"
      started_at: "2026-01-19T13:02:20Z"
      progress: "creating models"
      
    api_gateway:
      status: "waiting"
      waiting_for: ["user_service completion для shared types"]
      
    external_api:
      status: "pending"
      
    notifications:
      status: "pending"
  
  errors: []
  
  metrics:
    total_time_minutes: 5
    tokens_used: 15420
    estimated_cost_usd: 0.23
```

---

## Правила манифеста

### 1. Иммутабельность секций

| Секция | Когда меняется |
|--------|----------------|
| project | Только при создании |
| idea | Никогда после валидации |
| research | Только агентом-архитектором до валидации |
| constitution | Никогда (только через новую версию) |
| architecture | Только до валидации плана |
| flow | Автогенерация из architecture |
| validation | При прохождении gates |
| runtime | Постоянно во время выполнения |

### 2. Версионирование

При изменении архитектуры создаётся новый манифест:
```
manifests/
├── manifest-v1.yaml      # Первая версия
├── manifest-v2.yaml      # После изменений
└── manifest-current.yaml # Симлинк на актуальный
```

### 3. Валидация манифеста

Манифест должен быть валидным YAML и проходить schema validation:
- Все обязательные поля присутствуют
- Все block.id уникальны
- Все depends_on ссылаются на существующие блоки
- Нет циклических зависимостей
- Все contract paths существуют

---

## Жизненный цикл манифеста

```
[CREATION]
    │
    ▼
idea.raw ← Человек даёт идею
    │
    ▼
idea.interpreted ← Агент интерпретирует
    │
    ▼
idea.validated ← Человек подтверждает
    │
    ▼
research ← Агент исследует
    │
    ▼
architecture ← Агент декомпозирует
    │
    ▼
validation.plan = "pending" ← Ждёт валидации
    │
    ▼
[HUMAN REVIEW]
    │
    ├─→ approved → validation.plan = "approved"
    │
    └─→ rejected → возврат к architecture
    
[EXECUTION]
    │
    ▼
runtime.status = "running"
    │
    ▼
blocks выполняются по уровням
    │
    ▼
validation.gates проверяются
    │
    ▼
runtime.status = "completed"
```

---

## Связь с другими артефактами

```
manifest.yaml
    │
    ├── constitution.md          # Неизменные правила
    │
    ├── contracts/
    │   ├── auth.yaml            # Контракт блока auth
    │   ├── database.yaml        # Контракт блока database
    │   └── ...
    │
    └── results/
        ├── auth.json            # Результат блока auth
        ├── database.json        # Результат блока database
        └── ...
```

---

## Пример минимального манифеста

Для простого проекта из 2 блоков:

```yaml
version: "0.3"

project:
  id: "hello-api"
  name: "Hello World API"
  description: "Простой API для тестирования системы"

idea:
  raw: "Сделай простой API который отвечает hello world"
  interpreted: "REST API с одним endpoint GET / возвращающий JSON"
  validated: true

research:
  status: "completed"
  technical:
    - finding: "FastAPI — оптимальный выбор"
      rationale: "Минимальный boilerplate"

constitution:
  path: "./constitution.md"

architecture:
  total_levels: 1
  total_blocks: 2
  
  blocks:
    - id: "setup"
      name: "Project Setup"
      level: 1
      contract: "./contracts/setup.yaml"
      depends_on: []
      
    - id: "endpoint"
      name: "Hello Endpoint"
      level: 1
      contract: "./contracts/endpoint.yaml"
      depends_on:
        - block: "setup"
          required_output: "app_instance"

flow:
  diagram: |
    [setup] → [endpoint]

validation:
  plan:
    status: "approved"

runtime:
  status: "idle"
  blocks:
    setup:
      status: "pending"
    endpoint:
      status: "pending"
```
