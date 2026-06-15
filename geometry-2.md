# Практика: Geometry 2

## 1. Описание предметной области и сущностей
Система реализует паттерн Visitor для выполнения операций над геометрическими телами без модификации их классов. 
Основные сущности — абстрактный класс Body и его наследники (Ball, RectangularCuboid, Cylinder, CompoundBody), представляющие различные геометрические фигуры в трехмерном пространстве. Интерфейс IVisitor определяет методы для обработки каждой фигуры, а конкретные посетители (BoundingBoxVisitor и BoxifyVisitor) реализуют различные операции: вычисление ограничивающего параллелепипеда и замену фигур на их упрощенные прямоугольные представления соответственно.
Класс Body содержит свойство Position и абстрактный метод Accept для реализации двойной диспетчеризации. Каждая конкретная фигура переопределяет Accept, вызывая соответствующий метод Visit у посетителя.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Интерфейсы
    class IVisitor {
        <<interface>>
        +Body Visit(Ball ball)
        +Body Visit(RectangularCuboid cuboid)
        +Body Visit(Cylinder cylinder)
        +Body Visit(CompoundBody compoundBody)
    }
    
    %% Абстрактный класс
    class Body {
        <<abstract>>
        +Vector3 Position
        +Accept(IVisitor visitor)* Body
    }
    
    %% Конкретные классы фигур
    class Ball {
        +double Radius
        +Accept(IVisitor visitor) Body
    }
    
    class RectangularCuboid {
        +double SizeX
        +double SizeY
        +double SizeZ
        +Accept(IVisitor visitor) Body
    }
    
    class Cylinder {
        +double Radius
        +double SizeZ
        +Accept(IVisitor visitor) Body
    }
    
    class CompoundBody {
        +IReadOnlyList~Body~ Parts
        +ReplaceParts(IReadOnlyList~Body~)
        +Accept(IVisitor visitor) Body
    }
    
    %% Посетители
    class BoundingBoxVisitor {
        +Body Visit(Ball ball)
        +Body Visit(RectangularCuboid cuboid)
        +Body Visit(Cylinder cylinder)
        +Body Visit(CompoundBody compoundBody)
    }
    
    class BoxifyVisitor {
        +Body Visit(Ball ball)
        +Body Visit(RectangularCuboid cuboid)
        +Body Visit(Cylinder cylinder)
        +Body Visit(CompoundBody compoundBody)
    }
    
    %% Вспомогательные классы
    class Vector3 {
        +double X
        +double Y
        +double Z
    }
    
    %% Реализация интерфейса IVisitor
    IVisitor <|.. BoundingBoxVisitor : implements
    IVisitor <|.. BoxifyVisitor : implements
    
    %% Наследование от Body
    Body <|-- Ball : extends
    Body <|-- RectangularCuboid : extends
    Body <|-- Cylinder : extends
    Body <|-- CompoundBody : extends
    
    %% Агрегация CompoundBody с частями
    CompoundBody o-- Body : contains parts
    
    %% КОМПОЗИЦИЯ Body с Vector3 (исправлено!)
    Body *-- Vector3 : position
    
    %% Зависимости Accept от IVisitor
    Body ..> IVisitor : uses in Accept
    Ball ..> IVisitor : calls Visit
    RectangularCuboid ..> IVisitor : calls Visit
    Cylinder ..> IVisitor : calls Visit
    CompoundBody ..> IVisitor : calls Visit
    
    %% Зависимости посетителей от фигур
    BoundingBoxVisitor ..> Ball : visits
    BoundingBoxVisitor ..> RectangularCuboid : visits
    BoundingBoxVisitor ..> Cylinder : visits
    BoundingBoxVisitor ..> CompoundBody : visits
    
    BoxifyVisitor ..> Ball : visits
    BoxifyVisitor ..> RectangularCuboid : visits
    BoxifyVisitor ..> Cylinder : visits
    BoxifyVisitor ..> CompoundBody : visits
    
    %% BoundingBoxVisitor создает RectangularCuboid
    BoundingBoxVisitor ..> RectangularCuboid : creates
    
    %% BoxifyVisitor использует BoundingBoxVisitor
    BoxifyVisitor ..> BoundingBoxVisitor : uses
```