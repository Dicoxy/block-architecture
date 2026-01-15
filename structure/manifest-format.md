# Формат манифеста проекта

```yaml
project:
  name: "Project Name"
  version: "0.1.0"
  description: "Описание проекта"
  
blocks:
  - auth
  - user_profile
  - api_gateway
  - data_storage
  - notifications

flow:
  # Линейные связи
  auth -> api_gateway
  
  # Множественные входы
  api_gateway + auth -> user_profile
  
  # Разветвления
  api_gateway -> data_storage
  api_gateway -> notifications
  
  # Объединение
  data_storage + notifications -> report_generator
```

## Визуализация flow

Манифест должен позволять построить граф:

```
         ┌─────────────┐
         │    auth     │
         └──────┬──────┘
                │
                ▼
         ┌─────────────┐
         │ api_gateway │
         └──────┬──────┘
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
   ┌────────┐ ┌────────┐ ┌───────────┐
   │ user   │ │ data   │ │ notifi-   │
   │ profile│ │ storage│ │ cations   │
   └────────┘ └───┬────┘ └─────┬─────┘
                  │            │
                  └──────┬─────┘
                         ▼
                ┌────────────────┐
                │report_generator│
                └────────────────┘
```

## Статусы блоков в runtime

```yaml
runtime:
  auth:
    status: green  # green | red | waiting
    last_result: "token_abc123"
    timestamp: "2026-01-15T12:00:00Z"
    
  api_gateway:
    status: green
    last_result: {data: [...]}
    timestamp: "2026-01-15T12:00:01Z"
    
  data_storage:
    status: red
    error: "Connection timeout"
    timestamp: "2026-01-15T12:00:02Z"
```
