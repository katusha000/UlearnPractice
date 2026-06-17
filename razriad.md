# Практика: Контрольный разряд

## 1. Описание предметной области и сущностей
Система вычисления контрольных разрядов для различных стандартов нумерации (UPC, ISBN-10, алгоритм Луна). Контрольный разряд — это дополнительная цифра, вычисляемая по остальным цифрам номера, которая используется для автоматического обнаружения ошибок при ручном вводе или считывании номеров сканерами.
Сущности:

    ControlDigitAlgo - главный статический класс, предоставляющий публичный API для расчёта контрольных цифр по трём алгоритмам:
        Upc() - для штрих-кодов товаров (Universal Product Code)
        Isbn10() - для книжных номеров (International Standard Book Number)
        Luhn() - для банковских карт, IMEI и других идентификаторов
    Extensions - статический класс методов-расширений, инкапсулирующий общую вспомогательную логику:
        GetDigitsReversed() - извлечение цифр из числа справа налево
        SumWithWeights() - суммирование цифр с применением весовых коэффициентов
        GetCheckDigitMod10() - вычисление контрольной цифры по модулю 10
        AlternateWeights() - генерация чередующихся весов (3,1,3,1...)

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    class ControlDigitAlgo {
        <<static>>
        +static Upc(long number) int
        +static Isbn10(long number) char
        +static Luhn(long number) int
    }

    class Extensions {
        <<static>>
        +static GetDigitsReversed(long) IEnumerable~int~
        +static SumWithWeights(IEnumerable~int~, IEnumerable~int~) int
        +static GetCheckDigitMod10(int) int
        +static AlternateWeights(int, int, int) IEnumerable~int~
    }

    %% Зависимость: ControlDigitAlgo использует методы расширения из Extensions
    ControlDigitAlgo ..> Extensions : использует методы расширения
```