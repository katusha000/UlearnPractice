# Практика: GraphViz

## 1. Описание предметной области и сущностей
Система предназначена для построения графов в формате DOT (Graphviz) с использованием Fluent API - паттерна, позволяющего создавать объекты через цепочку вызовов методов, читаемую как естественный язык.
Точка входа - статический класс DotGraphBuilder, предоставляющий фабричные методы DirectedGraph() и UndirectedGraph() для создания направленных и ненаправленных графов.
Основной конструктор/строитель - GraphBuilder, который хранит ссылку на объект Graph и предоставляет методы AddNode(), AddEdge() и Build(). Методы AddNode() и AddEdge() возвращают специализированные билдеры (NodeBuilder и EdgeBuilder), которые наследуются от GraphBuilder, что позволяет продолжать цепочку вызовов без дублирования кода.
Паттерн Builder с конфигурацией: методы .With() принимают лямбда-выражения, настраивающие атрибуты через объекты NodeConfig и EdgeConfig. Эти конфигурационные классы наследуются от абстрактного ConfigBuilder, в котором реализованы общие атрибуты (Color, FontSize, Label). Специфичные атрибуты (Shape для узлов, Weight для рёбер) добавлены в дочерних классах.
Ключевая особенность Fluent API: метод .With() возвращает базовый тип GraphBuilder, у которого нет метода .With(), что предотвращает повторный вызов конфигурации для того же узла/ребра и обеспечивает типобезопасность цепочки.
Доменная модель (Graph, GraphNode, GraphEdge) отделена от API-слоя. Форматирование в DOT-строку делегировано классу DotFormatWriter, который корректно экранирует идентификаторы согласно спецификации Graphviz.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Точка входа
    class DotGraphBuilder {
        <<static>>
        +static DirectedGraph(string name) GraphBuilder
        +static UndirectedGraph(string name) GraphBuilder
    }

    %% Основные билдеры
    class GraphBuilder {
        -Graph graph
        +GraphBuilder(string name, bool directed)
        +virtual AddNode(string name) NodeBuilder
        +virtual AddEdge(string from, string to) EdgeBuilder
        +virtual Build() string
        #internal CommitNode(string, Dictionary) void
        #internal CommitEdge(string, string, Dictionary) void
    }

    class NodeBuilder {
        -string name
        -NodeConfig config
        -bool committed
        +NodeBuilder(GraphBuilder, string)
        +With(Action~NodeConfig~) GraphBuilder
        +override AddNode(string) NodeBuilder
        +override AddEdge(string, string) EdgeBuilder
        +override Build() string
    }

    class EdgeBuilder {
        -string from
        -string to
        -EdgeConfig config
        -bool committed
        +EdgeBuilder(GraphBuilder, string, string)
        +With(Action~EdgeConfig~) GraphBuilder
        +override AddNode(string) NodeBuilder
        +override AddEdge(string, string) EdgeBuilder
        +override Build() string
    }

    %% Конфигурации
    class ConfigBuilder~T~ {
        <<abstract>>
        #Dictionary~string, string~ Attributes
        +Color(string) T
        +FontSize(int) T
        +Label(string) T
    }

    class NodeConfig {
        +Shape(NodeShape) NodeConfig
    }

    class EdgeConfig {
        +Weight(double) EdgeConfig
    }

    %% Enum
    class NodeShape {
        <<enumeration>>
        Box
        Ellipse
        Circle
        Point
        Diamond
    }

    %% Существующие классы
    class Graph {
        -List~GraphEdge~ edges
        -Dictionary~string, GraphNode~ nodes
        +string GraphName
        +bool Directed
        +bool Strict
        +IEnumerable~GraphNode~ Nodes
        +IEnumerable~GraphEdge~ Edges
        +AddNode(string) GraphNode
        +AddEdge(string, string) GraphEdge
        +ToDotFormat() string
    }

    class GraphNode {
        +Dictionary~string, string~ Attributes
        +string Name
        +GraphNode(string name)
    }

    class GraphEdge {
        +Dictionary~string, string~ Attributes
        +string SourceNode
        +string DestinationNode
        +bool Directed
        +GraphEdge(string, string, bool)
    }

    class DotFormatWriter {
        -TextWriter writer
        +DotFormatWriter(TextWriter)
        +Write(Graph) void
        +Write(GraphNode) void
        +Write(GraphEdge) void
        +WriteAttributes(IReadOnlyDictionary) void
        +static EscapeId(string) string
    }

    class DotFormatExtensions {
        <<static>>
        +static ToDotFormat(Graph) string
    }

    %% ===== СВЯЗИ =====

    %% Наследование (<|--)
    ConfigBuilder~T~ <|-- NodeConfig
    ConfigBuilder~T~ <|-- EdgeConfig
    GraphBuilder <|-- NodeBuilder : extends
    GraphBuilder <|-- EdgeBuilder : extends

    %% Композиция (*--) 
    Graph *-- GraphNode : nodes
    Graph *-- GraphEdge : edges

    %% Агрегация (o--) 
    GraphBuilder o-- Graph : graph

    %% Зависимость (..>) 
    DotGraphBuilder ..> GraphBuilder : creates
    GraphBuilder ..> NodeBuilder : creates
    GraphBuilder ..> EdgeBuilder : creates
    NodeBuilder ..> NodeConfig : creates
    EdgeBuilder ..> EdgeConfig : creates
    NodeBuilder ..> NodeShape : uses
    DotFormatExtensions ..> DotFormatWriter : creates
    DotFormatWriter ..> Graph : uses
    DotFormatWriter ..> GraphNode : uses
    DotFormatWriter ..> GraphEdge : uses
```