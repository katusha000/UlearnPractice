# Практика: Robots

## 1. Описание предметной области и сущностей
Система моделирует взаимодействие между модулями принятия решений и исполнительными механизмами в автономных устройствах.
Ключевые компоненты:

    Планировщик (Planner) - компонент, отвечающий за формирование инструкций на основе текущего состояния.
    Исполнитель (Executor) - модуль, реализующий полученные инструкции и возвращающий результат выполнения.
    Инструкция (Instruction) - абстрактное представление действия, которое может быть выполнено.
    Навигационная инструкция (NavigationInstruction) - команда перемещения с указанием конечной позиции.
    Тактическая инструкция (TacticalInstruction) - расширенная навигационная команда с дополнительными параметрами поведения.
    Стратегический планировщик и Инженерный планировщик - специализированные реализации логики принятия решений.
    Мобильный исполнитель и Боевой исполнитель - конкретные механизмы выполнения команд.
    Автономный агент - основной объект, объединяющий планировщик и исполнитель через обобщённый интерфейс.
    Позиционный вектор - структура для хранения координат на плоскости.

Система использует ковариантность в интерфейсе планировщика (out T) и контравариантность в интерфейсе исполнителя (in T), что позволяет гибко комбинировать компоненты с разными уровнями специализации инструкций.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction LR

    class IInstruction {
        <<interface>>
    }

    class INavigationInstruction {
        <<interface>>
        +PositionVector Destination
    }

    class ITacticalInstruction {
        <<interface>>
        +bool StealthActive
    }

    class IPlanner~out T~ {
        <<interface>>
        +GenerateNextStep() T
    }

    class IExecutor~in T~ {
        <<interface>>
        +Run(T instruction) string
    }

    class TacticalPlanner {
        -int _stepCounter
        +GenerateNextStep() CombatDirective
    }

    class EngineeringPlanner {
        -int _phaseIndex
        +GenerateNextStep() BuildDirective
    }

    class MobileExecutor {
        +Run(INavigationInstruction move) string
    }

    class CombatExecutor {
        +Run(ITacticalInstruction engage) string
    }

    class AutonomousAgent~TInstruction~ {
        -IPlanner~TInstruction~ _decisionMaker
        -IExecutor~TInstruction~ _actionHandler
        +AutonomousAgent(IPlanner~TInstruction~, IExecutor~TInstruction~)
        +Operate(int cycles) IEnumerable~string~
    }

    class CombatDirective {
        +PositionVector Destination
        +bool StealthActive
        +CreateFromPhase(int phase)$ CombatDirective
    }

    class BuildDirective {
        +PositionVector Destination
        +bool ActivateStructure
        +CreateFromPhase(int phase)$ BuildDirective
    }

    class PositionVector {
        +double X
        +double Y
    }

    IInstruction <|-- INavigationInstruction : базовая навигация
    INavigationInstruction <|-- ITacticalInstruction : тактическое расширение
    ITacticalInstruction <|.. CombatDirective : боевая директива
    INavigationInstruction <|.. BuildDirective : строительная директива
    IPlanner~CombatDirective~ <|.. TacticalPlanner : тактическое планирование
    IPlanner~BuildDirective~ <|.. EngineeringPlanner : инженерное планирование
    IExecutor~INavigationInstruction~ <|.. MobileExecutor : мобильное исполнение
    IExecutor~ITacticalInstruction~ <|.. CombatExecutor : боевое исполнение
    AutonomousAgent~TInstruction~ o-- IPlanner~TInstruction~ : использует планировщик
    AutonomousAgent~TInstruction~ o-- IExecutor~TInstruction~ : делегирует исполнение
    CombatDirective --> PositionVector : целевая позиция
    BuildDirective --> PositionVector : целевая позиция
    ```