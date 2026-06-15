# Практика: [Название практики]

## 1. Описание предметной области и сущностей
Система моделирует взаимодействие игрока с объектами на игровой карте в компьютерной игре. 
Основные сущности представляют различные типы игровых объектов: Dwelling (жилище), Mine (шахта), Creeps (крипы), Wolves (волки) и ResourcePile (куча ресурсов). Для обеспечения гибкости и расширяемости выделены три интерфейса: IOwnable (объекты, которыми можно владеть), IFightable (объекты с армией для сражений) и ICollectible (объекты с сокровищами для сбора).
Класс Player отвечает за действия игрока: проверку возможности победы над армией (CanBeat), сбор сокровищ (Consume) и обработку смерти (Die). Статический класс Interaction координирует взаимодействие между игроком и объектами карты, используя только интерфейсы вместо конкретных классов.
## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Интерфейсы
    class IOwnable {
        <<interface>>
        +int Owner
    }
    
    class IFightable {
        <<interface>>
        +Army Army
    }
    
    class ICollectible {
        <<interface>>
        +Treasure Treasure
    }
    
    %% Классы объектов на карте
    class Dwelling {
        +int Owner
    }
    
    class Mine {
        +int Owner
        +Army Army
        +Treasure Treasure
    }
    
    class Creeps {
        +Army Army
        +Treasure Treasure
    }
    
    class Wolves {
        +Army Army
    }
    
    class ResourcePile {
        +Treasure Treasure
    }
    
    %% Вспомогательные классы
    class Player {
        +int Gold
        +bool Dead
        +int Id
        +bool CanBeat(Army)
        +void Consume(Treasure)
        +void Die()
    }
    
    class Interaction {
        +static void Make(Player, object)
    }
    
    class Army {
        +int Power
    }
    
    class Treasure {
        +int Amount
    }
    
    %% Реализация интерфейсов (пунктирная с треугольником)
    IOwnable <|.. Dwelling : implements
    IOwnable <|.. Mine : implements
    IFightable <|.. Mine : implements
    IFightable <|.. Creeps : implements
    IFightable <|.. Wolves : implements
    ICollectible <|.. Mine : implements
    ICollectible <|.. Creeps : implements
    ICollectible <|.. ResourcePile : implements
    
    %% Агрегация (классы хранят объекты в полях - пустой ромб)
    Mine o-- Army : contains
    Mine o-- Treasure : contains
    Creeps o-- Army : contains
    Creeps o-- Treasure : contains
    Wolves o-- Army : contains
    ResourcePile o-- Treasure : contains
    
    %% Зависимость (временное использование - пунктирная стрелка)
    Player ..> Army : uses in CanBeat
    Player ..> Treasure : uses in Consume
    Interaction ..> Player : uses
    Interaction ..> IOwnable : uses
    Interaction ..> IFightable : uses
    Interaction ..> ICollectible : uses
    ```