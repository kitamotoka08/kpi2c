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
