# Контракт блока

Формат описания входов/выходов блока.

## Базовая структура

```yaml
block:
  id: "unique_id"
  name: "Человекочитаемое название"
  level: 1  # уровень в иерархии
  
  description: |
    Что делает этот блок (для человека, не для агента)
  
  input:
    - name: "input_name"
      type: "string | number | object | array | file"
      required: true
      from_block: "block_id"  # если зависит от другого блока
      schema: {}  # если type: object
      
  output:
    success:
      type: "object"
      schema:
        result: "тип"
    error:
      type: "object"  
      schema:
        code: "integer"
        message: "string"
        
  validation:
    rule: "output.success XOR output.error"
    criteria:
      - "описание критерия успеха 1"
      - "описание критерия успеха 2"
      
  config:
    max_retries: 3
    timeout_minutes: 30
    escalate_on_fail: true
```

## Типы данных

| Тип | Описание |
|-----|----------|
| string | Текст |
| number | Число |
| boolean | true/false |
| object | Структура с полями |
| array | Список |
| file | Файл (путь или содержимое) |
| any | Любой тип (избегать) |

## Пример: блок транскрипции

```yaml
block:
  id: "transcription"
  name: "Транскрипция голоса"
  level: 2
  
  description: |
    Преобразует голосовое сообщение в текст
  
  input:
    - name: "audio"
      type: "file"
      required: true
      from_block: "voice_receiver"
      format: "ogg, mp3, wav"
      
  output:
    success:
      type: "object"
      schema:
        text: "string"
        confidence: "number"
        language: "string"
    error:
      type: "object"
      schema:
        code: "integer"
        message: "string"
        
  validation:
    rule: "output.success.text.length > 0"
    criteria:
      - "Текст не пустой"
      - "Язык определён"
      
  config:
    max_retries: 2
    timeout_minutes: 5
    escalate_on_fail: false
```
