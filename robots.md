# Практика: Robots

## 1. Описание предметной области и сущностей
Система моделирует архитектуру роботов с разделением ответственности между модулем принятия решений (AI) и исполнительным устройством (Device). Интерфейс IRobotAI<out TCommand> ковариантен и отвечает за генерацию команд, а IDevice<in TCommand> контравариантен и выполняет их исполнение. Универсальный класс Robot<TCommand> объединяет эти компоненты через dependency injection, обеспечивая гибкое комбинирование различных AI (ShooterAI, BuilderAI) и устройств (Mover, ShooterMover) благодаря вариативности обобщённых типов.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    %% Интерфейсы
    class IRobotAI~out TCommand~ {
        <<interface>>
        +TCommand GetCommand()
    }

    class IDevice~in TCommand~ {
        <<interface>>
        +string ExecuteCommand(TCommand command)
    }

    %% Интерфейсы
    class IMoveCommand {
        <<interface>>
        +Point Destination
    }

    class IShooterMoveCommand {
        <<interface>>
        +Point Destination
        +bool ShouldHide
    }

    %% Generic классы
    class RobotAI~TCommand~ {
        <<abstract>>
        #GetCommand()* TCommand
    }

    class Device~TCommand~ {
        <<abstract>>
        #ExecuteCommand(TCommand)* string
    }

    %% AI
    class ShooterAI {
        -int counter
        +GetCommand() IShooterMoveCommand
    }

    class BuilderAI {
        -int counter
        +GetCommand() IMoveCommand
    }

    %% Devices
    class Mover {
        +ExecuteCommand(IMoveCommand) string
    }

    class ShooterMover {
        +ExecuteCommand(IShooterMoveCommand) string
    }

    %% Generic Робот
    class Robot~TCommand~ {
        -IRobotAI~TCommand~ _ai
        -IDevice~TCommand~ _device
        +Robot(IRobotAI~TCommand~, IDevice~TCommand~)
        +Start(int steps) IEnumerable~string~
    }

    %% Фабричный класс
    class Robot {
        <<static>>
        +static Create~TCommand~(IRobotAI~TCommand~, IDevice~TCommand~) Robot~TCommand~
    }

    %% Классы команд
    class ShooterCommand {
        +Point Destination
        +bool ShouldHide
        +static ForCounter(int counter) IShooterMoveCommand
    }

    class BuilderCommand {
        +Point Destination
        +static ForCounter(int counter) IMoveCommand
    }

    %% Вспомогательные классы
    class Point {
        +int X
        +int Y
    }

    %% Свзяи

    %% Реализация интерфейсов
    IRobotAI~TCommand~ <|.. RobotAI~TCommand~
    IDevice~TCommand~ <|.. Device~TCommand~

    %% Наследование интерфейсов
    IMoveCommand <|-- IShooterMoveCommand : extends

    %% Наследование абстрактных классов
    RobotAI~TCommand~ <|-- ShooterAI
    RobotAI~TCommand~ <|-- BuilderAI
    Device~TCommand~ <|-- Mover
    Device~TCommand~ <|-- ShooterMover

    %% Реализация команд
    IShooterMoveCommand <|.. ShooterCommand
    IMoveCommand <|.. BuilderCommand

    %% Агрегация в Robot
    Robot~TCommand~ o-- IRobotAI~TCommand~ : использует AI
    Robot~TCommand~ o-- IDevice~TCommand~ : использует Device

    %% Зависимости
    Robot ..> Robot : фабрика создаёт
    ShooterAI ..> ShooterCommand : создаёт
    BuilderAI ..> BuilderCommand : создаёт
    ShooterCommand --> Point : содержит
    BuilderCommand --> Point : содержит
    ```