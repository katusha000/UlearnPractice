# Практика: Robots

## 1. Описание предметной области и сущностей
Система моделирует работу автоматизированного склада, где задачи генерируются координирующим модулем и выполняются физическими механизмами. Архитектура построена на принципе разделения генерации задач и их исполнения, что позволяет независимо развивать логику планирования и аппаратную часть.
Ключевые абстракции:
Базовый контракт IOperation описывает любое действие, которое может быть выполнено на складе. От него наследуется ITransferOperation - специализированный контракт для операций перемещения грузов между зонами хранения, содержащий целевую локацию. Далее расширяется до IPriorityOperation - операции с повышенным приоритетом, где добавляется флаг срочности выполнения.
Интерфейс ICoordinator с ковариантным параметром типа отвечает за генерацию последовательности операций. Ковариантность позволяет использовать координатор, производящий базовые операции перемещения, в контексте, ожидающем операции с приоритетом. Интерфейс IMechanism с контравариантным параметром описывает исполнительный механизм, способный обрабатывать операции определённого типа. Контравариантность даёт возможность механизму, работающему с базовыми операциями перемещения, принимать и операции с приоритетом.
Реализации:
WarehouseCoordinator и InventoryCoordinator - два типа координаторов для разных режимов работы склада (основное хранение и инвентаризация). ConveyorMechanism и RoboticArm - исполнительные устройства (конвейерная лента и роботизированная рука).
AutomatedUnit - универсальный модуль, объединяющий координатор и механизм через обобщённый интерфейс. TransferOperation и ProcessingOperation - конкретные классы операций с параметрами. Location - структура, описывающая позицию на складе (стеллаж, уровень).

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    %% Базовые интерфейсы операций
    class IOperation {
        <<interface>>
    }

    class ITransferOperation {
        <<interface>>
        +Location TargetLocation
    }

    class IPriorityOperation {
        <<interface>>
        +bool IsUrgent
    }

    %% Интерфейсы с вариативностью
    class ICoordinator~out T~ {
        <<interface>>
        +GenerateNextOperation() T
    }

    class IMechanism~in T~ {
        <<interface>>
        +Execute(T operation) string
    }

    %% Конкретные координаторы
    class WarehouseCoordinator {
        -int _sequenceIndex
        +GenerateNextOperation() TransferOperation
    }

    class InventoryCoordinator {
        -int _phaseNumber
        +GenerateNextOperation() ProcessingOperation
    }

    %% Конкретные механизмы
    class ConveyorMechanism {
        +Execute(ITransferOperation transfer) string
    }

    class RoboticArm {
        +Execute(IPriorityOperation priority) string
    }

    %% Универсальный модуль
    class AutomatedUnit~TOperation~ {
        -ICoordinator~TOperation~ coordinator
        -IMechanism~TOperation~ mechanism
        +AutomatedUnit(ICoordinator~TOperation~, IMechanism~TOperation~)
        +Run(int cycles) IEnumerable~string~
    }

    %% Классы данных операций
    class TransferOperation {
        +Location TargetLocation
        +bool IsUrgent
        +CreateForPhase(int phase)$ TransferOperation
    }

    class ProcessingOperation {
        +Location TargetLocation
        +bool RequiresVerification
        +CreateForPhase(int phase)$ ProcessingOperation
    }

    %% Структура локации
    class Location {
        +int RackNumber
        +int ShelfLevel
    }

    %% Связи

    %% Наследование интерфейсов
    IOperation <|-- ITransferOperation
    ITransferOperation <|-- IPriorityOperation

    %% Реализация интерфейсов координаторами
    ICoordinator~TransferOperation~ <|.. WarehouseCoordinator
    ICoordinator~ProcessingOperation~ <|.. InventoryCoordinator

    %% Реализация интерфейсов механизмами
    IMechanism~ITransferOperation~ <|.. ConveyorMechanism
    IMechanism~IPriorityOperation~ <|.. RoboticArm

    %% Реализация операций
    IPriorityOperation <|.. TransferOperation
    ITransferOperation <|.. ProcessingOperation

    %% Ассоциация компонентов модуля
    AutomatedUnit~TOperation~ --> ICoordinator~TOperation~ : координатор
    AutomatedUnit~TOperation~ --> IMechanism~TOperation~ : механизм

    %% Связи операций с локацией
    TransferOperation --> Location : целевая позиция
    ProcessingOperation --> Location : целевая позиция
    ```