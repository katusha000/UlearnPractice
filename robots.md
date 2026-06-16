# Практика: Robots

## 1. Описание предметной области и сущностей
Система моделирует архитектуру роботов с разделением ответственности между AI (выработка команд) и Device (исполнение команд).
Основные сущности: RobotAI - абстрактный класс для AI, вырабатывающего команды; Device - абстрактный класс для устройств, исполняющих команды. Конкретные реализации: ShooterAI и BuilderAI для разных типов роботов, Mover и ShooterMover для управления движением.
Интерфейс IRobotAI ковариантен (только производит команды), IDevice контравариантен (только потребляет команды). Это позволяет использовать Mover (работающий с IMoveCommand) для робота с IShooterMoveCommand благодаря контравариации, и BuilderAI (возвращающий IMoveCommand) благодаря ковариации.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Интерфейсы с вариативностью
    class IRobotAI {
        <<interface>>
        <<out TCommand>>
        +TCommand GetCommand()
    }
    
    class IDevice {
        <<interface>>
        <<in TCommand>>
        +string ExecuteCommand(TCommand command)
    }
    
    class IMoveCommand {
        <<interface>>
        +Point Destination
    }
    
    class IShooterMoveCommand {
        <<interface>>
        +Point Destination
        +bool ShouldHide
    }
    
    %% Абстрактные generic классы
    class RobotAI {
        <<abstract>>
        <<TCommand>>
        +GetCommand()* TCommand
    }
    
    class Device {
        <<abstract>>
        <<TCommand>>
        +ExecuteCommand(TCommand)* string
    }
    
    %% Конкретные реализации AI
    class ShooterAI {
        -int counter
        +GetCommand() IShooterMoveCommand
    }
    
    class BuilderAI {
        -int counter
        +GetCommand() IMoveCommand
    }
    
    %% Конкретные реализации Device
    class Mover {
        +ExecuteCommand(IMoveCommand) string
    }
    
    class ShooterMover {
        +ExecuteCommand(IShooterMoveCommand) string
    }
    
    %% Generic класс Robot
    class Robot {
        <<TCommand>>
        -IRobotAI~TCommand~ _ai
        -IDevice~TCommand~ _device
        +Robot(IRobotAI, IDevice)
        +Start(int) IEnumerable~string~
        +static Create(IRobotAI, IDevice) Robot~TCommand~
    }
    
    %% Классы команд
    class Point {
        +int X
        +int Y
    }
    
    class ShooterCommand {
        +Point Destination
        +bool ShouldHide
        +static ForCounter(int) IShooterMoveCommand
    }
    
    class BuilderCommand {
        +Point Destination
        +static ForCounter(int) IMoveCommand
    }
    
    %% Реализация интерфейсов
    IRobotAI <|.. RobotAI : implements
    IDevice <|.. Device : implements
    IMoveCommand <|-- IShooterMoveCommand : extends
    IMoveCommand <|.. ShooterCommand : implements
    IMoveCommand <|.. BuilderCommand : implements
    IShooterMoveCommand <|.. ShooterCommand : implements
    
    %% Наследование
    RobotAI <|-- ShooterAI : extends
    RobotAI <|-- BuilderAI : extends
    Device <|-- Mover : extends
    Device <|-- ShooterMover : extends
    
    %% Агрегация
    Robot o-- IRobotAI : uses ai
    Robot o-- IDevice : uses device
    
    %% Композиция 
    ShooterCommand *-- Point : destination
    BuilderCommand *-- Point : destination
    
    %% Зависимости
    ShooterAI ..> ShooterCommand : creates
    BuilderAI ..> BuilderCommand : creates
    ShooterMover ..> IShooterMoveCommand : uses
    Mover ..> IMoveCommand : uses
    ```