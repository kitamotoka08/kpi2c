Сгенерувати ЕР Діаграму, використовуючи ці критерії: 
    User - Користувач:
    user_id - унікальний айді користувача (PK, UUID)
    username - нікнейм користувача (string)
    email - електронна пошта (string)
    registered_at - дата реєстрації (date)
    Title - Тайтл (Аніме / Манга):
    title_id - унікальний айді тайтла (PK, UUID)
    name - назва твору (string)
    type - тип твору: аніме, манґа, ранобе тощо (string)
    episodes_or_chapters - загальна кількість епізодів або розділів (int)
    status - статус виходу: триває, завершено, анонсовано (string)
    Genre - Жанр:
    genre_id - унікальний айді жанру (PK, UUID)
    genre_name - назва жанру (string)
    Studio - Студія:
    studio_id - унікальний айді студії (PK, UUID)
    studio_name - назва студії-виробника (string)
    country - країна походження (string)
    Review - Відгук:
    review_id - унікальний айді відгуку (PK, UUID)
    user_id - посилання на користувача, який залишив відгук (FK, UUID)
    title_id - посилання на тайтл (FK, UUID)
    rating - оцінка від користувача (int)
    comment - текст відгуку (string)
Та ці відносини:
    Studio 1:N Title
    Title N:M Genre
    User 1:N Review
    Title 1:N Review
повинно бути написано на мові Mermaid
