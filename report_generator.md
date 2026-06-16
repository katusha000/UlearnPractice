# Практика: Report generator

## 1. Описание предметной области и сущностей
Система предназначена для генерации отчётов о статистике погодных измерений (температуры и влажности) с использованием различных форматов вывода и методов вычисления показателей.
Абстрактный класс ReportMaker реализует шаблонный метод MakeReport, определяющий общий алгоритм формирования отчёта, но делегирует конкретные детали форматирования (заголовки, списки, элементы) и вычисления статистики подклассам. Четыре конкретных класса-наследника (MeanAndStdHtmlReportMaker, MedianMarkdownReportMaker, MeanAndStdMarkdownReportMaker, MedianHtmlReportMaker) комбинируют два типа форматирования (HTML и Markdown) с двумя типами статистики (среднее со стандартным отклонением и медиана). Класс Measurement хранит данные измерений, MeanAndStd представляет результат вычисления среднего и отклонения, а ReportMakerHelper предоставляет удобные статические методы для создания отчётов без явного использования конкретных классов.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Абстрактный класс
    class ReportMaker {
        <<abstract>>
        #string Caption*
        #string MakeCaption(string caption)*
        #string BeginList()*
        #string EndList()*
        #string MakeItem(string valueType, string entry)*
        #object MakeStatistics(IEnumerable~double~ data)*
        +string MakeReport(IEnumerable~Measurement~ measurements)
    }
    
    %% Конкретные реализации
    class MeanAndStdHtmlReportMaker {
        #string Caption
        #string MakeCaption(string caption)
        #string BeginList()
        #string EndList()
        #string MakeItem(string valueType, string entry)
        #object MakeStatistics(IEnumerable~double~ data)
    }
    
    class MedianMarkdownReportMaker {
        #string Caption
        #string MakeCaption(string caption)
        #string BeginList()
        #string EndList()
        #string MakeItem(string valueType, string entry)
        #object MakeStatistics(IEnumerable~double~ data)
    }
    
    class MeanAndStdMarkdownReportMaker {
        #string Caption
        #string MakeCaption(string caption)
        #string BeginList()
        #string EndList()
        #string MakeItem(string valueType, string entry)
        #object MakeStatistics(IEnumerable~double~ data)
    }
    
    class MedianHtmlReportMaker {
        #string Caption
        #string MakeCaption(string caption)
        #string BeginList()
        #string EndList()
        #string MakeItem(string valueType, string entry)
        #object MakeStatistics(IEnumerable~double~ data)
    }
    
    %% Вспомогательные классы
    class ReportMakerHelper {
        +static string MeanAndStdHtmlReport(IEnumerable~Measurement~ data)
        +static string MedianMarkdownReport(IEnumerable~Measurement~ data)
        +static string MeanAndStdMarkdownReport(IEnumerable~Measurement~ data)
        +static string MedianHtmlReport(IEnumerable~Measurement~ data)
    }
    
    class Measurement {
        +double Temperature
        +double Humidity
    }
    
    class MeanAndStd {
        +double Mean
        +double Std
        +string ToString()
    }
    
    %% Наследование 
    ReportMaker <|-- MeanAndStdHtmlReportMaker : extends
    ReportMaker <|-- MedianMarkdownReportMaker : extends
    ReportMaker <|-- MeanAndStdMarkdownReportMaker : extends
    ReportMaker <|-- MedianHtmlReportMaker : extends
    
    %% Зависимости
    ReportMaker ..> Measurement : uses
    ReportMaker ..> MeanAndStd : creates
    ReportMakerHelper ..> ReportMaker : uses
    ReportMakerHelper ..> Measurement : uses
    MeanAndStdHtmlReportMaker ..> MeanAndStd : creates
    MeanAndStdMarkdownReportMaker ..> MeanAndStd : creates
    ```