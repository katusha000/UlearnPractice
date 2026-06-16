# Практика: TaxiOrder

## 1. Описание предметной области и сущностей
Система автоматизирует обработку заявок в службе такси, управляя жизненным циклом заказа от момента создания до завершения поездки.
Архитектурный подход: Решение построено на принципах предметно-ориентированного проектирования (DDD), где бизнес-логика инкапсулирована в доменных моделях, а техническая 
Компоненты:

    Базовые абстракции (Entity<TKey>, ValueType<T>) - обеспечивают общую функциональность для сущностей и объектов-значений, реализуя семантику сравнения и идентичности.
    Объекты-значения (PersonName, Address, Car) - неизменяемые объекты, определяемые своими атрибутами, а не идентичностью. Сравниваются по значению всех полей.
    Сущности (Driver, TaxiOrder) - объекты с уникальной идентичностью (Id), которые проходят через различные состояния в процессе жизненного цикла.
    Агрегат TaxiOrder - корневая сущность, инкапсулирующая правила перехода между статусами заказа и обеспечивающая согласованность данных.
    Сервисный слой (TaxiApi) - координирует взаимодействие между компонентами, делегируя бизнес-правила доменным объектам.
    Репозиторий (DriversRepository) - абстрагирует доступ к данным о водителях, предоставляя интерфейс для поиска по идентификатору.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Инфраструктурные компоненты
    class ValueType~T~ {
        <<abstract>>
        +Equals(T other) bool
        +GetHashCode() int
        +ToString() string
    }

    class Entity~TId~ {
        <<abstract>>
        +Id : TId
        +Equals(object) bool
        +GetHashCode() int
    }

    %% Value Objects
    class PersonName {
        +FirstName : string
        +LastName : string
        +ToString() string
    }

    class Address {
        +Street : string
        +Building : string
        +ToString() string
    }

    class Car {
        +Model : string
        +Color : string
        +PlateNumber : string
    }

    %% Entities
    class Driver {
        +Id : int
        +Name : PersonName
        +Car : Car
    }

    class TaxiOrder {
        +Id : int
        +ClientName : PersonName
        +Start : Address
        +Destination : Address
        +Driver : Driver
        +Status : TaxiOrderStatus
        +CreationTime : DateTime
        +DriverAssignmentTime : DateTime?
        +StartRideTime : DateTime?
        +FinishRideTime : DateTime?
        +CancelTime : DateTime?
        +Create(...) TaxiOrder
        +UpdateDestination(Address) void
        +AssignDriver(Driver, DateTime) void
        +UnassignDriver(DateTime) void
        +StartRide(DateTime) void
        +FinishRide(DateTime) void
        +Cancel(DateTime) void
    }

    class TaxiOrderStatus {
        <<enumeration>>
        WaitingForDriver
        WaitingCarArrival
        InProgress
        Finished
        Canceled
    }

    %% API layer
    class ITaxiApi~T~ {
        <<interface>>
        +CreateOrder(...) T
        +UpdateDestination(T, string, string) void
        +AssignDriver(T, int) void
        +UnassignDriver(T) void
        +Cancel(T) void
        +StartRide(T) void
        +FinishRide(T) void
        +GetDriverFullInfo(T) string
        +GetShortOrderInfo(T) string
    }

    class TaxiApi {
        -driversRepo : DriversRepository
        -currentTime : Func~DateTime~
        -idCounter : int
        +CreateOrder(...) TaxiOrder
        +UpdateDestination(TaxiOrder, string, string) void
        +AssignDriver(TaxiOrder, int) void
        +UnassignDriver(TaxiOrder) void
        +Cancel(TaxiOrder) void
        +StartRide(TaxiOrder) void
        +FinishRide(TaxiOrder) void
        +GetDriverFullInfo(TaxiOrder) string
        +GetShortOrderInfo(TaxiOrder) string
    }

    class DriversRepository {
        +GetDriverById(int) Driver
    }

    %% ===== СВЯЗИ =====

    %% 1. НАСЛЕДОВАНИЕ (<|--)
    ValueType~T~ <|-- PersonName
    ValueType~T~ <|-- Address
    ValueType~T~ <|-- Car
    Entity~TId~ <|-- Driver
    Entity~TId~ <|-- TaxiOrder

    %% 2. РЕАЛИЗАЦИЯ ИНТЕРФЕЙСА (<|..)
    ITaxiApi~T~ <|.. TaxiApi

    %% 3. КОМПОЗИЦИЯ (*--)
    Driver *-- PersonName : Name
    Driver *-- Car : Car
    TaxiOrder *-- PersonName : ClientName
    TaxiOrder *-- Address : Start
    TaxiOrder *-- Address : Destination

    %% 4. АГРЕГАЦИЯ (o--)
    TaxiApi o-- DriversRepository : driversRepo
    TaxiOrder o-- Driver : Driver

    %% 5. АССОЦИАЦИЯ (-->)
    TaxiOrder --> TaxiOrderStatus : Status

    %% 6. ЗАВИСИМОСТЬ (..>)
    TaxiApi ..> TaxiOrder : creates
    DriversRepository ..> Driver : creates
    DriversRepository ..> PersonName : creates
    DriversRepository ..> Car : creates
    TaxiApi ..> Address : creates
```