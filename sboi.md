# Практика: Сбои

## 1. Описание предметной области и сущностей
Система реализует анализ исторических данных об отказах технического оборудования с целью выявления устройств, подверженных критическим сбоям в заданный период времени.
Архитектурные слои:
Доменный слой (Domain Layer):

    Device - агрегатный корень, представляющий единицу оборудования с уникальным идентификатором и наименованием.
    Failure - сущность события отказа, инкапсулирующая тип неисправности, временную метку и ссылку на устройство. Содержит методы бизнес-логики для оценки серьезности (IsFailureSerious) и временного сравнения (IsEarlierThan).
    FailureType - типизированное перечисление классификации отказов (аппаратурные, программные, сетевые).

Сервисный слой (Service Layer):

    ReportMaker - сервисный класс, реализующий use-case "Формирование отчета о проблемных устройствах". Анализирует коллекцию сбоев, применяет критерии фильтрации (тип, дата) и возвращает имена устройств.

Слой совместимости (Legacy Layer):

    Common - утилитарный класс, сохраняющий устаревшие методы для обратной совместимости со старым API. Используется методом FindDevicesFailedBeforeDateObsolete.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB
    
    %% Доменный слой 
    class Device {
        <<entity>>
        +int DeviceId
        +string Name
        +Device(int id, string name)
    }
    
    class Failure {
        <<entity>>
        +FailureType Type
        +int Day
        +int Month
        +int Year
        +int DeviceId
        +IsFailureSerious() bool
        +IsEarlierThan(DateTime date) bool
    }
    
    class FailureType {
        <<enumeration>>
        UnexpectedShutdown = 0
        ShortNonResponding = 1
        HardwareFailures = 2
        ConnectionProblems = 3
    }
    
    %% Сервисный слой 
    class ReportMaker {
        <<service>>
        +FindDevicesFailedBeforeDate(devices: List~Device~, failures: List~Failure~, beforeDate: DateTime) List~string~$
        +FindDevicesFailedBeforeDateObsolete(day: int, month: int, year: int, failureTypes: int[], deviceId: int[], times: object[][], devices: List~Dictionary~string, object~~) List~string~$
    }
    
    %% Слой совместимости 
    class Common {
        <<helper>>
        +IsFailureSerious(failureType: int) int$
        +Earlier(v: object[], day: int, month: int, year: int) int$
    }
    
    %% Группировка по слоям 
    subgraph Domain
        Device
        Failure
        FailureType
    end
    
    subgraph Service
        ReportMaker
    end
    
    subgraph Legacy
        Common
    end
    
    %% Связи
    
    %% Доменные связи
    Failure --> FailureType : классифицирует тип отказа
    Failure --> Device : ссылается на устройство через DeviceId
    
    %% Сервисные связи
    ReportMaker ..> Failure : анализирует коллекцию сбоев
    ReportMaker ..> Device : извлекает имена проблемных устройств
    ReportMaker ..> Common : делегирует устаревшую логику
    ```