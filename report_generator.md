# Практика: Report generator

## 1. Описание предметной области и сущностей
Система формирует отчёты на основе коллекции погодных измерений (температура и влажность). Изначально в коде присутствовал комбинаторный взрыв классов: для каждой комбинации формата вывода и типа статистики создавался отдельный класс-наследник (MeanAndStdHtmlReportMaker, MedianMarkdownReportMaker и т.д.), что нарушало принцип единственной ответственности (SRP) и принцип открытости/закрытости (OCP).
Рефакторинг заменил наследование на композицию стратегий (паттерн Strategy). Теперь ответственность разделена на две независимые оси изменений:

    Форматирование отчёта — как выглядят заголовки, списки и элементы (HTML или Markdown).
    Вычисление статистики — какой показатель рассчитывается по данным (среднее с отклонением или медиана).

Ключевые компоненты:

    IReportFormatter - интерфейс, описывающий операции форматирования: создание заголовка, открытие/закрытие списка, оформление элемента.
    IStatisticsCalculator - интерфейс, отвечающий за вычисление статистического показателя по набору данных.
    HtmlFormatter и MarkdownFormatter - конкретные реализации форматтера.
    MeanAndStdCalculator и MedianCalculator - конкретные реализации калькулятора.
    ReportMaker - центральный класс, принимающий через конструктор обе стратегии. Больше не является абстрактным и не требует наследования.
    Measurement - структура данных измерения (температура, влажность).
    IStatisticResult, MeanAndStd, Median - результаты вычислений.
    ReportMakerHelper - фасад со статическими методами для удобного создания отчётов без явного конструирования стратегий.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    %% Стратегии форматирования
    class IReportFormatter {
        <<interface>>
        +MakeCaption(string text) string
        +OpenList() string
        +CloseList() string
        +FormatEntry(string kind, string value) string
    }

    class HtmlFormatter {
        +MakeCaption(string text) string
        +OpenList() string
        +CloseList() string
        +FormatEntry(string kind, string value) string
    }

    class MarkdownFormatter {
        +MakeCaption(string text) string
        +OpenList() string
        +CloseList() string
        +FormatEntry(string kind, string value) string
    }

    %% Стратегии вычисления статистики
    class IStatisticsCalculator {
        <<interface>>
        +Compute(IEnumerable~double~ values) IStatisticResult
    }

    class MeanAndStdCalculator {
        +Compute(IEnumerable~double~ values) IStatisticResult
    }

    class MedianCalculator {
        +Compute(IEnumerable~double~ values) IStatisticResult
    }

    %% Результаты вычислений
    class IStatisticResult {
        <<interface>>
        +ToString() string
    }

    class MeanAndStd {
        +double Mean
        +double StdDev
        +ToString() string
    }

    class Median {
        +double Value
        +ToString() string
    }

    %% Данные и главный класс
    class Measurement {
        +double Temperature
        +double Humidity
    }

    class ReportMaker {
        -IReportFormatter formatter
        -IStatisticsCalculator calculator
        +ReportMaker(IReportFormatter, IStatisticsCalculator)
        +BuildReport(IEnumerable~Measurement~ data) string
    }

    %% Фасад
    class ReportMakerHelper {
        +static string BuildMeanHtml(IEnumerable~Measurement~ data)
        +static string BuildMeanMarkdown(IEnumerable~Measurement~ data)
        +static string BuildMedianHtml(IEnumerable~Measurement~ data)
        +static string BuildMedianMarkdown(IEnumerable~Measurement~ data)
    }

    %% Связи

    %% Реализация интерфейсов
    IReportFormatter <|.. HtmlFormatter : реализует HTML-вывод
    IReportFormatter <|.. MarkdownFormatter : реализует Markdown-вывод
    IStatisticsCalculator <|.. MeanAndStdCalculator : вычисляет среднее
    IStatisticsCalculator <|.. MedianCalculator : вычисляет медиану
    IStatisticResult <|.. MeanAndStd : результат среднего
    IStatisticResult <|.. Median : результат медианы

    %% Агрегация — стратегии передаются через конструктор
    ReportMaker o-- IReportFormatter : использует форматтер
    ReportMaker o-- IStatisticsCalculator : использует калькулятор

    %% Зависимости — временное использование
    ReportMaker ..> Measurement : обрабатывает данные
    ReportMaker ..> IStatisticResult : получает результат
    MeanAndStdCalculator ..> MeanAndStd : создаёт
    MedianCalculator ..> Median : создаёт
    ReportMakerHelper ..> ReportMaker : создаёт
    ReportMakerHelper ..> HtmlFormatter : создаёт
    ReportMakerHelper ..> MarkdownFormatter : создаёт
    ReportMakerHelper ..> MeanAndStdCalculator : создаёт
    ReportMakerHelper ..> MedianCalculator : создаёт
    ReportMakerHelper ..> Measurement : принимает данные
    ```