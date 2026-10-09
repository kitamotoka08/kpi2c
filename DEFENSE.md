# Чому цей домен?

- Наявність зв'язку M:N
- Чіткий розподіл на незалежні сутності
- Наявність історії взаємодій через відгуки з фіксацією оцінок.

# Чому 3NF

- Немає залежності між неключовими полями, всі атрибути залежать від первинного ключа.

# Синхронізація

| Атрибут | spec | erdiagram |
| :--- | :--- | :--- |
| User | user_id, username, email, registered_at | user_id, username, email, registered_at |
| Title | title_id, studio_id, name, type, episodes_or_chapters, status | title_id, studio_id, name, type, episodes_or_chapters, status |
| Genre | genre_id, genre_name | genre_id, genre_name |
| Studio | studio_id, studio_name, country | studio_id, studio_name, country |
| Review | review_id, user_id, title_id, rating, comment | review_id, user_id, title_id, rating, comment |
| Типи | PK/FK - UUID | PK/FK - UUID |
