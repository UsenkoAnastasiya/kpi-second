# spec.md — Бібліотека

## Намір

Описати дані бібліотеки: що в каталозі, які фізичні примірники є, хто бере книги і коли.

## Сутності та атрибути

- **Publisher**: publisher_id (PK), name, city
- **Author**: author_id (PK), full_name, birth_year
- **Genre**: genre_id (PK), name
- **Book** (книга як видання в каталозі): book_id (PK), isbn (унікальний), title, publication_year, publisher_id (FK)
- **BookCopy** (фізичний примірник): copy_id (PK), inventory_code (унікальний), status, book_id (FK)
- **Reader**: reader_id (PK), full_name, email (унікальний), registered_at
- **Loan** (асоціативна сутність Reader–BookCopy): loan_id (PK), reader_id (FK), copy_id (FK), loaned_at, due_at, returned_at (може бути порожнім)

## Зв'язки та кардинальності

- Publisher 1 — 0..N Book (видавець видає багато книг; книга має рівно одного видавця)
- Author N — M Book (книга має ≥1 автора; автор може мати 0..N книг) — чистий M:N, **без сполучної таблиці**
- Genre N — M Book (книга має 0..N жанрів) — чистий M:N
- Book 1 — 1..N BookCopy (кожна книга в каталозі має ≥1 примірник)
- Reader 1 — 0..N Loan
- BookCopy 1 — 0..N Loan
- Loan — асоціативна сутність, бо зв'язок «читач бере примірник» має власні атрибути (loaned_at, due_at, returned_at).

## Критерії прийняття

1. Усі сутності мають PK; усі зв'язки 1:N мають FK на боці «багато».
2. Усі id мають однаковий тип `string`; імена `<entity>_id`.
3. Модель у 3NF: немає атрибутів, що залежать не від ключа (напр. назва видавця не дублюється в Book; кількість примірників не зберігається, а обчислюється).
4. M:N Author–Book і Genre–Book змодельовано як зв'язки, без сутностей user_roles-типу.
5. Асоціативна сутність лише там, де зв'язок має власні атрибути (Loan).
6. Назви атрибутів у `model/library.mmd` збігаються зі spec символ у символ.
7. Жодного SQL/ORM.
8. Кардинальності в моделі точно відповідають розділу «Зв'язки»:
   «≥1» → `|{` / `|o`, «0..N» → `o{`. Зокрема Author–Book: книга має ≥1 автора (`}o--|{`).
9. Типи атрибутів: id та текстові поля — `string`; `birth_year`, `publication_year` — `int`;
   `registered_at`, `loaned_at`, `due_at`, `returned_at` — `date`.
10. Імена сутностей у моделі — UPPER_SNAKE_CASE, напр. `BOOK_COPY` (spec: BookCopy).
