# Оркестратор (LangGraph Engine)

Оркестратор — движок который исполняет манифест.

## Роль

Оркестратор читает утверждённый манифест и координирует выполнение блоков:
- Определяет порядок запуска
- Управляет зависимостями
- Запускает агентов параллельно где возможно
- Проверяет phase gates
- Собирает результаты
- Обновляет runtime состояние манифеста

## Почему LangGraph

LangGraph — фреймворк для stateful мульти-агентных систем от LangChain.

Преимущества:
- Граф состояний уже реализован
- Параллельное выполнение из коробки
- Checkpointing (можно остановить и продолжить)
- Условные переходы
- Интеграция с Claude, GPT, локальные модели

## Архитектура оркестратора

```
┌─────────────────────────────────────────────────────────────┐
│                      ОРКЕСТРАТОР                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Loader    │    │  Scheduler  │    │   Runner    │     │
│  │             │    │             │    │             │     │
│  │ - manifest  │ →  │ - порядок   │ →  │ - запуск    │     │
│  │ - contracts │    │ - parallel  │    │ - агентов   │     │
│  │ - constit.  │    │ - gates     │    │ - tools     │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                  │                  │             │
│         ▼                  ▼                  ▼             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    STATE GRAPH                       │   │
│  │                    (LangGraph)                       │   │
│  │                                                      │   │
│  │  [start] → [level_1] → [validate] → [level_2] → ... │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  Validator  │    │   State     │    │  Reporter   │     │
│  │             │    │   Manager   │    │             │     │
│  │ - contracts │    │ - runtime   │    │ - logs      │     │
│  │ - criteria  │    │ - results   │    │ - metrics   │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Компоненты

### 1. Loader

Загружает и валидирует артефакты:

```python
class Loader:
    def load_manifest(self, path: str) -> Manifest:
        """Загружает и валидирует манифест"""
        
    def load_constitution(self, path: str) -> Constitution:
        """Загружает constitution, проверяет checksum"""
        
    def load_contracts(self, manifest: Manifest) -> dict[str, Contract]:
        """Загружает все контракты блоков"""
        
    def validate_graph(self, manifest: Manifest) -> ValidationResult:
        """Проверяет что граф валиден (нет циклов, все ссылки есть)"""
```

### 2. Scheduler

Определяет порядок выполнения:

```python
class Scheduler:
    def build_execution_plan(self, manifest: Manifest) -> ExecutionPlan:
        """
        Строит план выполнения:
        - Топологическая сортировка
        - Группировка параллельных блоков
        - Phase gates между уровнями
        """
        
    def get_ready_blocks(self, state: RuntimeState) -> list[str]:
        """Возвращает блоки готовые к запуску (все зависимости green)"""
        
    def check_phase_gate(self, level: int, state: RuntimeState) -> bool:
        """Проверяет можно ли перейти к следующему уровню"""
```

### 3. Runner

Запускает агентов:

```python
class Runner:
    def create_agent(self, block_id: str, contract: Contract) -> Agent:
        """Создаёт агента для блока с нужными tools"""
        
    async def run_block(self, block_id: str, inputs: dict) -> BlockResult:
        """Запускает блок и возвращает результат"""
        
    async def run_parallel(self, block_ids: list[str]) -> dict[str, BlockResult]:
        """Запускает несколько блоков параллельно"""
```

### 4. Validator

Проверяет результаты:

```python
class Validator:
    def validate_result(self, block_id: str, result: Any, contract: Contract) -> ValidationResult:
        """
        Проверяет:
        - Структура соответствует output schema
        - Все success_criteria выполнены
        - Артефакты созданы
        """
        
    def validate_constitution(self, block_id: str, artifacts: list[str], constitution: Constitution) -> ValidationResult:
        """Проверяет что артефакты соответствуют constitution"""
```

### 5. State Manager

Управляет состоянием:

```python
class StateManager:
    def update_block_status(self, block_id: str, status: str, result: Any = None):
        """Обновляет статус блока в runtime"""
        
    def save_checkpoint(self) -> str:
        """Сохраняет checkpoint для возможности продолжения"""
        
    def load_checkpoint(self, checkpoint_id: str) -> RuntimeState:
        """Загружает состояние из checkpoint"""
```

## LangGraph State

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph

class ProjectState(TypedDict):
    # Загруженные артефакты
    manifest: dict
    constitution: dict
    contracts: dict[str, dict]
    
    # Runtime состояние
    current_level: int
    block_results: dict[str, Any]
    block_statuses: dict[str, str]  # pending | running | green | red
    
    # Метаданные
    errors: list[str]
    start_time: str
    tokens_used: int

# Создание графа
workflow = StateGraph(ProjectState)
```

## Flow выполнения

```python
# Псевдокод основного flow

async def orchestrate(manifest_path: str):
    # 1. Загрузка
    loader = Loader()
    manifest = loader.load_manifest(manifest_path)
    constitution = loader.load_constitution(manifest.constitution.path)
    contracts = loader.load_contracts(manifest)
    
    # 2. Валидация графа
    validation = loader.validate_graph(manifest)
    if not validation.ok:
        raise InvalidManifestError(validation.errors)
    
    # 3. Построение плана
    scheduler = Scheduler()
    plan = scheduler.build_execution_plan(manifest)
    
    # 4. Инициализация состояния
    state = ProjectState(
        manifest=manifest,
        constitution=constitution,
        contracts=contracts,
        current_level=1,
        block_results={},
        block_statuses={b.id: "pending" for b in manifest.blocks},
        errors=[],
        start_time=now(),
        tokens_used=0
    )
    
    # 5. Выполнение по уровням
    runner = Runner()
    validator = Validator()
    
    for level in plan.levels:
        # Получить готовые блоки
        ready = scheduler.get_ready_blocks(state)
        
        # Запустить параллельно
        results = await runner.run_parallel(ready)
        
        # Валидировать каждый результат
        for block_id, result in results.items():
            validation = validator.validate_result(
                block_id, 
                result, 
                contracts[block_id]
            )
            
            if validation.ok:
                state.block_statuses[block_id] = "green"
                state.block_results[block_id] = result
            else:
                state.block_statuses[block_id] = "red"
                state.errors.append(validation.error)
                # Retry или escalate
        
        # Проверить phase gate
        if not scheduler.check_phase_gate(level, state):
            break
    
    # 6. Финализация
    return state
```

## Phase Gates

Phase gates — контрольные точки между уровнями:

```yaml
# В манифесте
validation:
  gates:
    - after_level: 1
      type: "automatic"
      criteria: "all blocks green"
      
    - after_level: 2
      type: "human"
      criteria: "API responds to health check"
```

Типы gates:
- **automatic** — оркестратор проверяет сам
- **human** — ждёт подтверждения от человека

```python
async def check_phase_gate(self, gate: PhaseGate, state: RuntimeState) -> bool:
    if gate.type == "automatic":
        # Проверяем критерий программно
        return self.evaluate_criteria(gate.criteria, state)
    
    elif gate.type == "human":
        # Уведомляем человека и ждём
        await self.notify_human(gate)
        return await self.wait_for_approval(gate.id)
```

## Обработка ошибок

```python
class ErrorHandler:
    async def handle_block_error(self, block_id: str, error: Exception, state: RuntimeState):
        contract = state.contracts[block_id]
        
        if contract.execution.on_failure == "retry":
            if state.retry_count[block_id] < contract.execution.max_retries:
                await self.retry_block(block_id)
            else:
                await self.escalate(block_id, error)
                
        elif contract.execution.on_failure == "escalate":
            await self.escalate(block_id, error)
            
        elif contract.execution.on_failure == "skip":
            state.block_statuses[block_id] = "skipped"
            
    async def escalate(self, block_id: str, error: Exception):
        """Уведомляет человека о проблеме"""
        # Отправить уведомление
        # Ждать решения
```

## Checkpointing

Возможность остановить и продолжить:

```python
# Сохранение checkpoint
checkpoint_id = state_manager.save_checkpoint()
print(f"Saved checkpoint: {checkpoint_id}")

# Позже — продолжение
state = state_manager.load_checkpoint(checkpoint_id)
await orchestrate_from_state(state)
```

Это критично для:
- Долгих проектов
- Human-in-the-loop gates
- Восстановления после сбоев

## Метрики

Оркестратор собирает:

```python
metrics = {
    "total_time_seconds": 180,
    "blocks_completed": 5,
    "blocks_failed": 1,
    "tokens_used": 25000,
    "estimated_cost_usd": 0.38,
    "retries": 2,
    "human_interventions": 1
}
```
