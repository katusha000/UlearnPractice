# Практика: Дифференцирование

## 1. Описание предметной области и сущностей
Система символьного дифференцирования предназначена для автоматического вычисления производных математических функций, представленных в виде деревьев выражений LINQ Expressions.
Основная сущность Algebra содержит статический метод Differentiate, который принимает лямбда-выражение типа Expression и возвращает его производную. Метод рекурсивно обходит дерево выражений, применяя правила дифференцирования для различных типов узлов: констант (производная = 0), переменных (производная = 1), сумм (правило суммы), произведений (правило произведения), синусов и косинусов (цепное правило).
Классы Expression, BinaryExpression, UnaryExpression, MethodCallExpression представляют узлы дерева выражений. ExpressionType определяет тип операции (сложение, умножение, вызов метода и т.д.). Система выбрасывает ArgumentException с информативным сообщением при попытке дифференцировать неподдерживаемые функции (например, Math.Max) или синтаксические конструкции.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Основной класс
    class Algebra {
        <<static>>
        +static Expression~Func~double, double~~ Differentiate(Expression~Func~double, double~~ function)
        -static Expression DifferentiateExpression(Expression expr, ParameterExpression parameter)
    }
    
    %% Абстрактный класс Expression
    class Expression {
        <<abstract>>
        +ExpressionType NodeType
    }
    
    %% Классы выражений
    class BinaryExpression {
        +Expression Left
        +Expression Right
    }
    
    class UnaryExpression {
        +Expression Operand
    }
    
    class MethodCallExpression {
        +MethodInfo Method
        +IReadOnlyList~Expression~ Arguments
    }
    
    class MemberExpression {
        +MemberInfo Member
        +Expression Expression
    }
    
    class ConstantExpression {
        +object Value
    }
    
    class ParameterExpression {
        +string Name
        +Type Type
    }
    
    %% Перечисление
    class ExpressionType {
        <<enumeration>>
        Add
        Multiply
        Call
        Convert
        MemberAccess
        Constant
        Parameter
    }
    
    %% Класс Math
    class Math {
        <<static>>
        +static double Sin(double x)
        +static double Cos(double x)
    }
    
    %% Generic delegate
    class Func~T, TResult~ {
        <<delegate>>
    }
    
    %% НАСЛЕДОВАНИЕ
    Expression <|-- BinaryExpression : extends
    Expression <|-- UnaryExpression : extends
    Expression <|-- MethodCallExpression : extends
    Expression <|-- MemberExpression : extends
    Expression <|-- ConstantExpression : extends
    Expression <|-- ParameterExpression : extends
    
    %% КОМПОЗИЦИЯ
    BinaryExpression *-- Expression : contains Left/Right
    UnaryExpression *-- Expression : contains Operand
    MethodCallExpression *-- Expression : contains Arguments
    MemberExpression *-- Expression : contains Expression
    
    %% ЗАВИСИМОСТЬ
    Algebra ..> Expression : uses
    Algebra ..> ExpressionType : checks
    Algebra ..> Math : differentiates
    ExpressionType ..> BinaryExpression : describes type
    ExpressionType ..> UnaryExpression : describes type
    ExpressionType ..> MethodCallExpression : describes type
    ExpressionType ..> MemberExpression : describes type
    ```