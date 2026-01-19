# Манифест проекта

Полное описание проекта для агента-архитектора.

## Структура

```yaml
project:
  id: "unique_project_id"
  name: "Название проекта"
  version: "0.1.0"
  
  idea: |
    Исходная идея от человека.
    Текст как есть, без обработки.
    
  research:
    completed: true
    findings:
      - "Находка 1"
      - "Находка 2"
    competitors:
      - name: "Конкурент"
        url: "https://..."
        notes: "Что взять/избежать"
    tech_stack:
      recommended:
        - "технология 1"
        - "технология 2"
      
  architecture:
    levels: 3
    blocks:
      - id: "block_1"
        level: 1
        contract: "./contracts/block_1.yaml"
      - id: "block_2"
        level: 1  
        contract: "./contracts/block_2.yaml"
      - id: "block_3"
        level: 2
        contract: "./contracts/block_3.yaml"
        depends_on:
          - "block_1"
          - "block_2"
          
  flow:
    diagram: |
      [block_1] [block_2]
           ↘   ↙
          [block_3]
          
  validation:
    status: "approved"  # draft | pending | approved
    approved_by: "human"
    approved_at: "2026-01-15T12:00:00Z"
    notes: "Комментарии от человека"
    
  runtime:
    status: "running"  # idle | running | completed | failed
    blocks:
      block_1:
        status: "green"
        result: "..."
      block_2:
        status: "green"
        result: "..."
      block_3:
        status: "running"
        progress: "50%"
```

## Жизненный цикл манифеста

```
[idea] → [research] → [architecture] → [validation] → [runtime] → [completed]
            ↑              ↑               ↑
         агент          агент          человек
```

1. **idea** — человек даёт идею
2. **research** — агент исследует
3. **architecture** — агент декомпозирует
4. **validation** — человек утверждает
5. **runtime** — агенты исполняют
6. **completed** — результат готов
