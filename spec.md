# Атрибути:
* User - Користувач:
  *   user_id - унікальний айді користувача (PK, UUID)
  *   username - нікнейм користувача (string)
  *   email - електронна пошта (string)
  *   registered_at - дата реєстрації (date)
* Title - Тайтл (Аніме / Манга):
  *   title_id - унікальний айді тайтла (PK, UUID)
  *   studio_id - (optional, FK, UUID)
  *   name - назва твору (string)
  *   type - тип твору: аніме, манґа, ранобе тощо (string)
  *   episodes_or_chapters - загальна кількість епізодів або розділів (int)
  *   status - статус виходу: триває, завершено, анонсовано (string)
* Genre - Жанр:
  * genre_id - унікальний айді жанру (PK, UUID)
  * genre_name - назва жанру (string)
* UserListEntry - Список користувач:   
  * entry_id - унікальний айді запису (PK, UUID)
  * user_id - посилання на користувача (FK, UUID)
  * title_id - посилання на тайтл (FK, UUID)
  * watch_status - статус: планую, дивлюся, переглянуто, кинуто (string)
  * progress - кількість переглянутих епізодів / прочитаних глав (int)
* Studio - Студія:
  *   studio_id - унікальний айді студії (PK, UUID)
  *   studio_name - назва студії-виробника (string)
  *   country - країна походження (string)
* Review - Відгук:
  *   review_id - унікальний айді відгуку (PK, UUID)
  *   user_id - посилання на користувача, який залишив відгук (FK, UUID)
  *   title_id - посилання на тайтл (FK, UUID)
  *   rating - оцінка від користувача (int)
  *   comment - текст відгуку (string)

# Відносини:
* Studio |o--o{ Title
* Title }|--|{ Genre
* User ||--o{ Review
* Title ||--o{ Review
* User ||--o{ UserListEntry
* Title ||--o{ UserListEntry

# Критерії прийняття
* Нормалізація: Модель відповідає вимогам 3NF
* Ключова узгодженність: Усі PK та FK мають UUID тип даних
* Цілісність данних: Оцінка у відгуках має обмеження 1-5, а для манґи наявність студії необов'язкова.

# Gherkin-сценарії
```
Feature: Керування персональним списком тайтлів

  Scenario: Успішне оновлення прогресу та статусу перегляду
    Given Користувач авторизований у системі
    And Тайтл "Fullmetal Alchemist: Brotherhood" знаходиться у списку UserListEntry зі статусом watch_status = "дивлюся" та progress = 10
    When Користувач змінює progress на 11 та залишає watch_status = "дивлюся"
    Then Система зберігає progress = 11 та watch_status = "дивлюся" у сутності UserListEntry
    And Користувач бачить сповіщення про успішне оновлення

  Scenario: Автоматичне завершення при досягненні максимального епізоду
    Given Користувач авторизований у системі
    And Тайтл має episodes_or_chapters = 64, а запис у UserListEntry має watch_status = "дивлюся"
    When Користувач оновлює progress до 64
    Then Система автоматично змінює UserListEntry.watch_status на "переглянуто" без зміни Title.status
    And Система оновлює UserListEntry.progress до 64

  Scenario Outline: Помилка при введенні невалідного прогресу
    Given Користувач авторизований у системі
    And Тайтл має episodes_or_chapters = <max_episodes>, а поточний UserListEntry.progress = <current>
    When Користувач намагається встановити progress = <invalid_progress>
    Then Система відхиляє запит і залишає UserListEntry.progress = <current>
    And Система виводить повідомлення про помилку валідації діапазону прогресу

    Examples:
      | max_episodes | current | invalid_progress |
      | 12           | 5       | -1               |
      | 12           | 5       | 13               |
```

# EARS-вимоги під ці сценарії
* REQ-01 (Перевірка авторизації): WHEN неавторизований запит надходить на зміну або створення запису в UserListEntry, SYSTEM SHALL відхилити запит зі статусом 401 і не змінювати дані в базі.
* REQ-02 (Оновлення прогресу та статусу списку): WHEN авторизований користувач надсилає валідний progress (0 <= progress <= Title.episodes_or_chapters) для власного запису UserListEntry, SYSTEM SHALL оновити поля UserListEntry.progress та UserListEntry.watch_status і повернути статус успіху.
* REQ-03 (Автозміна статусу списку): WHILE значення UserListEntry.progress дорівнює Title.episodes_or_chapters, SYSTEM SHALL автоматично встановити UserListEntry.watch_status у значення "переглянуто" без зміни Title.status.
* REQ-04 (Валідація діапазону прогресу): WHEN введений progress менший за 0 або більший за Title.episodes_or_chapters, SYSTEM SHALL відхилити транзакцію та видати помилку валідації.

# Нефункціональні вимоги
* NFR-01 (Продуктивність / Час відгуку): Час відповіді сервера на запит оновлення запису UserListEntry за 95-м перцентилем (p95) не має перевищувати 250 мс при піковому навантаженні 200 RPS протягом 10 хвилин.
* NFR-02 (Навантаження та межі): Система має стабільно працювати в діапазоні від 100 до 3 000 одночасних активних користувачів із часткою помилок 5xx менш ніж 0.5%; при перевищенні 3 000 користувачів система повертає HTTP 429 замість падіння сервісу.
* NFR-03 (Безпека та автентифікація): 100% запитів на зміну UserListEntry мають перевіряти підпис JWT-токена та збіг user_id із власником запису; запити без валідного токена або до чужого запису відхиляються зі статусом 401 або 403.
* NFR-04 (Підтримувані пристрої та інтерфейс): Вебдодаток має коректно відображатися у Chrome 120+, Firefox 120+ і Safari 17+ на екранах шириною від 360px до 2560px без горизонтального скролу.
* NFR-05 (Мова та локалізація): Інтерфейс платформи та повідомлення про помилки мають бути українською мовою у кодуванні UTF-8.
* NFR-06 (Технологічний стек): Бекенд розгортається на базі Node.js із використанням реляційної бази даних PostgreSQL та ідентифікаторів типу UUID.

# Поза обсягом
* Інтеграція зі сторонніми стрімінговими сервисами для автоматичного трекінгу перегляду в реальному часі.
* Реалізація рекомендаційної системи на основі машинного навчання для підбору аніме за схожими вподобаннями.
* Внутрішній чат між користувачами та система приватних повідомлень.

# Матриця трасування
| Вимога ID | Сценарій Gherkin | Пов'язані UC | Статус / Обґрунтування |
|:--|:--|:--|
| REQ-01 | Усі сценарії UC-3 | UC-3, UC-4 | Спільна перевірка авторизації для всіх змін користувача. |
| REQ-02 | Сценарій 1, Сценарій 2 | UC-3 | Збереження валідних полів progress і watch_status. |
| REQ-03 | Сценарій 2 | UC-3 | Автоматичний перехід UserListEntry.watch_status у "переглянуто" |
| REQ-04| Scenario Outline (-1, 13) | UC-3 | Захист цілісності progress на нижній і верхній межах |

# Актори та цілі 
* Гість: переглядає каталог тайтлів, шукає за жанром або студією, читає відгуки.
* Зареєстрований користувач: веде власний список, оновлює статус перегляду та прогрес, залишає відгуки.

# Use-Case діаграма

```
@startuml
left to right direction

actor "Гість" as Guest
actor "Зареєстрований користувач" as User
actor "Модератор" as Moderator

User -|> Guest

rectangle "Трекер аніме та манґи" {
  usecase "Переглянути каталог тайтлів" as UC1
  usecase "Пошук за жанром або студією" as UC2
  usecase "Керувати персональним списком тайтлів" as UC3
  usecase "Залишити відгук та оцінку" as UC4
  usecase "Переглянути відгуки" as UC5
  usecase "Модерувати відгуки" as UC6
}

Guest --> UC1
Guest --> UC2
Guest --> UC5

User --> UC3
User --> UC4

Moderator --> UC5
Moderator --> UC6

UC2 ..> UC1 : <<include>>
UC4 ..> UC5 : <<extend>>
@enduml
```

# Завдання 3: Sequence-діаграми для сценаріїв UC-3

## Учасники та кроки сценаріїв

### Сценарій 1: Успішне оновлення прогресу та статусу перегляду
- Учасники: Користувач, ListController, ListService, UserListEntryRepository, TitleRepository.
- Кроки:
  1. Користувач змінює progress на 11 та залишає watch_status = "дивлюся" для тайтла у своєму списку.
  2. ListController перевіряє автентифікацію та передає запит у ListService.
  3. ListService отримує запис UserListEntry і перевіряє ліміт Title.episodes_or_chapters.
  4. Оскільки progress менший за максимуму та не виходить за межі, ListService оновлює UserListEntry через UserListEntryRepository і повертає результат.
  5. Користувач бачить підтвердження успішного оновлення.

### Сценарій 2: Автоматичне завершення при досягненні максимального епізоду
- Учасники: Користувач, ListController, ListService, UserListEntryRepository, TitleRepository.
- Кроки:
  1. Користувач оновлює progress до 64 (дорівнює Title.episodes_or_chapters).
  2. ListController делегує обробку у ListService.
  3. ListService звіряє progress із Title.episodes_or_chapters і виявляє досягнення максимуму.
  4. ListService автоматично встановлює UserListEntry.watch_status = "переглянуто" (не змінюючи Title.status) і зберігає зміни через UserListEntryRepository.
  5. Користувач отримує оновлений стан запису.

### Сценарій 3 (Scenario Outline): Помилка при введенні невалідного прогресу
- Учасники: Користувач, ListController, ListService, TitleRepository.
- Кроки:
  1. Користувач намагається встановити невалідне значення progress (наприклад, -1 або 13 при максимумі 12).
  2. ListController передає запит у ListService.
  3. ListService перевіряє межі (0 <= progress <= Title.episodes_or_chapters) і виявляє порушення.
  4. Запит відхиляється з помилкою валідації, дані в UserListEntry не змінюються.
  5. Користувач бачить повідомлення про помилку валідації.

## Sequence-діаграма 1: Успішне оновлення прогресу та статусу перегляду

```
@startuml
autonumber
actor "Користувач" as User
participant "ListController" as Controller
participant "ListService" as Service
participant "TitleRepository" as TitleRepo
participant "UserListEntryRepository" as EntryRepo

User -> Controller : updateListEntry(userId, titleId, progress=11, status="дивлюся")
activate Controller

Controller -> Service : updateProgress(userId, titleId, 11, "дивлюся")
activate Service

Service -> TitleRepo : findById(titleId)
activate TitleRepo
TitleRepo --> Service : Title(episodes_or_chapters=24, status="триває")
deactivate TitleRepo

Service -> EntryRepo : findByUserAndTitle(userId, titleId)
activate EntryRepo
EntryRepo --> Service : UserListEntry(progress=10, watch_status="дивлюся")
deactivate EntryRepo

alt progress валідний (0 <= progress <= episodes_or_chapters)
  Service -> EntryRepo : save(entryId, progress=11, watch_status="дивлюся")
  activate EntryRepo
  EntryRepo --> Service : updatedEntry
  deactivate EntryRepo
  Service --> Controller : 200 OK (updatedEntry)
  Controller --> User : підтвердження оновлення
else progress поза межами або запис не знайдено
  Service --> Controller : ValidationError
  Controller --> User : помилка валідації
end

deactivate Service
deactivate Controller
@enduml
```

# Sequence-діаграма 2: Автоматичне завершення при досягненні максимального епізоду

```
@startuml
autonumber
actor "Користувач" as User
participant "ListController" as Controller
participant "ListService" as Service
participant "TitleRepository" as TitleRepo
participant "UserListEntryRepository" as EntryRepo

User -> Controller : updateProgress(userId, titleId, progress=64)
activate Controller

Controller -> Service : handleProgressUpdate(userId, titleId, 64)
activate Service

Service -> TitleRepo : findById(titleId)
activate TitleRepo
TitleRepo --> Service : Title(episodes_or_chapters=64, status="триває")
deactivate TitleRepo

Service -> EntryRepo : findByUserAndTitle(userId, titleId)
activate EntryRepo
EntryRepo --> Service : UserListEntry(progress=63, watch_status="дивлюся")
deactivate EntryRepo

opt progress == Title.episodes_or_chapters
  Service -> Service : setWatchStatus("переглянуто")
end

Service -> EntryRepo : save(entryId, progress=64, watch_status="переглянуто")
activate EntryRepo
EntryRepo --> Service : savedEntry
deactivate EntryRepo

Service --> Controller : 200 OK (savedEntry)
Controller --> User : оновлений запис зі статусом "переглянуто"

deactivate Service
deactivate Controller
@enduml
```

# Sequence-діаграма 3: Помилка при введенні невалідного прогресу
```
@startuml
autonumber
actor "Користувач" as User
participant "ListController" as Controller
participant "ListService" as Service
participant "TitleRepository" as TitleRepo

User -> Controller : updateProgress(userId, titleId, invalidProgress)
activate Controller

Controller -> Service : validateAndUpdate(userId, titleId, invalidProgress)
activate Service

Service -> TitleRepo : findById(titleId)
activate TitleRepo
TitleRepo --> Service : Title(episodes_or_chapters=12)
deactivate TitleRepo

alt invalidProgress < 0
  Service --> Controller : Error("progress не може бути меншим за 0")
else invalidProgress > Title.episodes_or_chapters
  Service --> Controller : Error("progress перевищує кількість епізодів")
end

Controller --> User : 400 Bad Request (повідомлення про помилку)

deactivate Service
deactivate Controller
@enduml
```
