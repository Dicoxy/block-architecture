# Формат описания блока

```yaml
block:
  id: "unique_block_id"
  name: "Человекочитаемое название"
  description: "Что делает этот блок"
  
  input:
    - name: "input_name"
      type: "string | number | object | array"
      required: true
      schema:  # если type: object
        field1: type
        field2: type
        
  output:
    - name: "success"
      type: "object"
      schema:
        result: type
    - name: "error"
      type: "object"
      schema:
        code: integer
        message: string
        
  validation:
    rule: "Логическое правило проверки результата"
    test: "Как проверить что результат валидный"
    
  depends_on:
    - block_id_1
    - block_id_2
```

## Пример: Блок авторизации

```yaml
block:
  id: "auth"
  name: "Авторизация"
  description: "Проверяет credentials и выдаёт токен"
  
  input:
    - name: "credentials"
      type: "object"
      required: true
      schema:
        login: string
        password: string
        
  output:
    - name: "success"
      type: "object"
      schema:
        token: string
        expires_at: timestamp
    - name: "error"
      type: "object"
      schema:
        code: integer  # 401, 403, 500
        message: string
        
  validation:
    rule: "output.success XOR output.error"
    test: "token is valid JWT OR error.code in [401, 403, 500]"
    
  depends_on: []
```

## Пример: Блок с несколькими входами

```yaml
block:
  id: "report_generator"
  name: "Генератор отчётов"
  description: "Создаёт отчёт на основе данных пользователя и шаблона"
  
  input:
    - name: "user_data"
      type: "object"
      from_block: "user_profile"
    - name: "template"
      type: "object"
      from_block: "template_storage"
    - name: "config"
      type: "object"
      from_block: "settings"
        
  output:
    - name: "report"
      type: "object"
      schema:
        content: string
        format: string  # pdf, html, markdown
        generated_at: timestamp
        
  depends_on:
    - user_profile
    - template_storage
    - settings
```
