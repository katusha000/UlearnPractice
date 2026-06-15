# Практика: Сбои

## 1. Описание предметной области и сущностей
Система предназначена для анализа сбоев в работе устройств и формирования отчетов о критических отказах. 
Основная сущность Device представляет устройство с уникальным идентификатором и именем. Сущность Failure описывает факт сбоя: содержит тип отказа (FailureType), дату возникновения и ссылку на устройство. Типы сбоев варьируются от неожиданных отключений до аппаратных неисправностей.
Класс ReportMaker отвечает за формирование списка устройств, в которых произошли критические сбои (типы 0 и 2 — четные значения) до указанной даты. Для этого он анализирует коллекцию сбоев, проверяя их серьезность и дату возникновения, затем сопоставляет их с устройствами и возвращает имена проблемных устройств. Вспомогательный класс Common содержит устаревшие методы для обратной совместимости.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    class Device {
        +int DeviceId
        +string Name
    }
    
    class Failure {
        +FailureType Type
        +int Day
        +int Month
        +int Year
        +int DeviceId
        +bool IsFailureSerious()
        +bool IsEarlierThan(DateTime date)
    }
    
    class FailureType {
        <<enumeration>>
        UnexpectedShutdown = 0
        ShortNonResponding = 1
        HardwareFailures = 2
        ConnectionProblems = 3
    }
    
    class ReportMaker {
        +static List~string~ FindDevicesFailedBeforeDate(List~Device~ devices, List~Failure~ failures, DateTime beforeDate)
        +static List~string~ FindDevicesFailedBeforeDateObsolete(int day, int month, int year, int[] failureTypes, int[] deviceId, object[][] times, List~Dictionary~string, object~~ devices)
    }
    
    class Common {
        +static int IsFailureSerious(int failureType)
        +static int Earlier(object[] v, int day, int month, int year)
    }
    
    Failure ..> FailureType : type
    ReportMaker ..> Device : uses
    ReportMaker ..> Failure : uses
    ReportMaker ..> Common : uses
    ```