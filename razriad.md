# Практика: Контрольный разряд

## 1. Описание предметной области и сущностей
Система вычисления контрольных разрядов для различных стандартов нумерации (UPC, ISBN-10, алгоритм Луна). Контрольный разряд - это дополнительная цифра, вычисляемая по остальным цифрам номера, которая используется для автоматического обнаружения ошибок при ручном вводе или считывании номеров сканерами.
Сущности:

    ControlDigitAlgo - главный класс, предоставляющий публичный API для расчёта контрольных цифр по трём алгоритмам:
        Upc() - для штрих-кодов товаров (Universal Product Code)
        Isbn10() - для книжных номеров (International Standard Book Number)
        Luhn() - для банковских карт, IMEI и других идентификаторов
    Extensions - класс методов расширений, инкапсулирующий общую вспомогательную логику:
        GetDigitsReversed() - извлечение цифр из числа справа налево
        SumWithWeights() - суммирование цифр с применением весовых коэффициентов
        GetCheckDigitMod10() - вычисление контрольной цифры по модулю 10
        AlternateWeights() - генерация чередующихся весов (3,1,3,1...)

Класс ControlDigitAlgo делегирует повторяющиеся операции методам из Extensions, что позволяет избежать дублирования кода и упрощает тестирование. Каждый алгоритм использует специфичные веса и формулы, но опирается на общие примитивы для извлечения цифр и вычисления контрольного разряда.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Классы
    class ControlDigitAlgo {
        <<static>>
        +static Upc(long number) int
        +static Isbn10(long number) char
        +static Luhn(long number) int
    }

    %% Методы расширения 
    class Extensions {
        <<static>>
        +static GetDigitsReversed(long) IEnumerable~int~
        +static SumWithWeights(IEnumerable~int~, IEnumerable~int~) int
        +static GetCheckDigitMod10(int) int
        +static AlternateWeights(int, int, int) IEnumerable~int~
    }

    %% Алгоритм UPC 
    class UpcAlgorithm {
        <<utility>>
        Описание: Universal Product Code
        Используется: для штрих-кодов
        Веса: чередуются 3, 1, 3, 1...
        Формула: (10 - (sum % 10)) % 10
    }

    %% Алгоритм ISBN-10
    class Isbn10Algorithm {
        <<utility>>
        Описание: International Standard Book Number
        Используется: для книг
        Веса: от 2 до 10
        Формула: (11 - (sum % 11)) % 11
        Результат: 0-9 или 'X' (для 10)
    }

    %% Алгоритм Luhn
    class LuhnAlgorithm {
        <<utility>>
        Описание: Luhn algorithm (mod 10)
        Используется: для карт, IMEI
        Логика: удвоение каждой 2-й цифры
        Если > 9: вычитаем 9
    }

    %% Вспомогательные операции
    class DigitExtraction {
        <<utility>>
        +ExtractDigits(number) IEnumerable~int~
        +Reverse(number) IEnumerable~int~
        +DoubleDigit(digit) int
    }

    class WeightCalculation {
        <<utility>>
        +AlternateWeights(first, second) IEnumerable~int~
        +SequentialWeights(start, count) IEnumerable~int~
        +ApplyWeights(digits, weights) IEnumerable~int~
    }

    class CheckDigitComputation {
        <<utility>>
        +Mod10(sum) int
        +Mod11(sum) char
        +LuhnMod10(sum) int
    }

    %% Связи

    %% Зависимости (..>)
    ControlDigitAlgo ..> Extensions : использует методы расширения
    ControlDigitAlgo ..> UpcAlgorithm : implements
    ControlDigitAlgo ..> Isbn10Algorithm : implements
    ControlDigitAlgo ..> LuhnAlgorithm : implements
    
    Extensions ..> DigitExtraction : provides
    Extensions ..> WeightCalculation : provides
    Extensions ..> CheckDigitComputation : provides
    
    UpcAlgorithm ..> WeightCalculation : uses alternate weights
    UpcAlgorithm ..> CheckDigitComputation : uses mod10
    
    Isbn10Algorithm ..> WeightCalculation : uses sequential weights
    Isbn10Algorithm ..> CheckDigitComputation : uses mod11
    
    LuhnAlgorithm ..> DigitExtraction : uses doubling
    LuhnAlgorithm ..> CheckDigitComputation : uses mod10

    %% Агрегация (общее использование)
    Extensions o-- ControlDigitAlgo : extends
```