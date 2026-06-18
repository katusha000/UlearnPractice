# Практика: Report generator

## 1. Описание предметной области и сущностей
Система решает задачу формирования аналитических сводок по метеорологическим наблюдениям (температура, влажность). Исходная реализация страдала от комбинаторного взрыва наследников: каждый подкласс ReportMaker одновременно кодировал и способ визуального представления, и математический метод анализа, что приводило к дублированию логики и нарушению принципа единственной ответственности.
Рефакторинг применяет паттерн Strategy, разделяя две ортогональные оси изменчивости:

    Представление (Presentation) - отвечает за синтаксис выходного документа (разметка заголовков, списков, элементов).
    Аналитика (Analytics) - отвечает за математическую обработку числовой выборки.

Компоненты системы:

    IOutputRenderer - контракт для модулей визуального оформления. Определяет операции рендеринга заголовка, секции списка и отдельной строки.
    IMetricAnalyzer - контракт для модулей статистического анализа. Принимает числовую выборку и возвращает структурированный результат.
    WebRenderer и TextRenderer - конкретные реализации рендерера для HTML и Markdown соответственно.
    AverageDeviationAnalyzer и MedianAnalyzer - конкретные реализации аналитических модулей.
    ReportComposer - центральный класс-композитор, собирающий итоговый документ из двух стратегий, переданных через конструктор.
    WeatherRecord - структура исходных данных (температура, влажность).
    IAnalysisResult, AverageWithDeviation, MedianValue - иерархия результатов анализа.
    QuickReport - фасад, упрощающий клиентский код за счёт скрытия конструирования стратегий.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction LR

    %% Слой представления
    subgraph Presentation["Слой представления"]
        class IOutputRenderer {
            <<interface>>
            +RenderHeader(string title) string
            +OpenSection() string
            +AddRow(string label, string value) string
            +CloseSection() string
        }

        class WebRenderer {
            +RenderHeader(string title) string
            +OpenSection() string
            +AddRow(string label, string value) string
            +CloseSection() string
        }

        class TextRenderer {
            +RenderHeader(string title) string
            +OpenSection() string
            +AddRow(string label, string value) string
            +CloseSection() string
        }
    end

    %% Слой аналитики
    subgraph Analytics["Слой аналитики"]
        class IMetricAnalyzer {
            <<interface>>
            +Analyze(IEnumerable~double~ sample) IAnalysisResult
        }

        class AverageDeviationAnalyzer {
            +Analyze(IEnumerable~double~ sample) IAnalysisResult
        }

        class MedianAnalyzer {
            +Analyze(IEnumerable~double~ sample) IAnalysisResult
        }
    end

    %% Результаты анализа
    subgraph Results["Результаты"]
        class IAnalysisResult {
            <<interface>>
            +string AsText()
        }

        class AverageWithDeviation {
            +double Average
            +double Deviation
            +string AsText()
        }

        class MedianValue {
            +double Value
            +string AsText()
        }
    end

    %% Данные
    class WeatherRecord {
        +double Temperature
        +double Humidity
    }

    class ReportComposer {
        -IOutputRenderer renderer
        -IMetricAnalyzer analyzer
        +ReportComposer(IOutputRenderer, IMetricAnalyzer)
        +Compose(IEnumerable~WeatherRecord~ records) string
    }

    class QuickReport {
        +static string GenerateAverageWeb(IEnumerable~WeatherRecord~ records)
        +static string GenerateAverageText(IEnumerable~WeatherRecord~ records)
        +static string GenerateMedianWeb(IEnumerable~WeatherRecord~ records)
        +static string GenerateMedianText(IEnumerable~WeatherRecord~ records)
    }

    %% Связи

    %% Реализация интерфейсов представления
    IOutputRenderer <|.. WebRenderer
    IOutputRenderer <|.. TextRenderer

    %% Реализация интерфейсов аналитики
    IMetricAnalyzer <|.. AverageDeviationAnalyzer
    IMetricAnalyzer <|.. MedianAnalyzer

    %% Реализация результатов
    IAnalysisResult <|.. AverageWithDeviation
    IAnalysisResult <|.. MedianValue

    %% Агрегация стратегий в композиторе
    ReportComposer o-- IOutputRenderer : стратегия представления
    ReportComposer o-- IMetricAnalyzer : стратегия анализа

    %% Зависимости композитора
    ReportComposer ..> WeatherRecord : обрабатывает записи
    ReportComposer ..> IAnalysisResult : использует результат

    %% Создание результатов аналитикой
    AverageDeviationAnalyzer ..> AverageWithDeviation : формирует
    MedianAnalyzer ..> MedianValue : формирует

    %% Фасад создаёт компоненты
    QuickReport ..> ReportComposer : собирает
    QuickReport ..> WebRenderer : инициализирует
    QuickReport ..> TextRenderer : инициализирует
    QuickReport ..> AverageDeviationAnalyzer : инициализирует
    QuickReport ..> MedianAnalyzer : инициализирует
    QuickReport ..> WeatherRecord : принимает данные
    ```