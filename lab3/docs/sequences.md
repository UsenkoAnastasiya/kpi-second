# Sequence-діаграми для UC-2 «Видати примірник»

## S-02: Успішна видача вільного примірника

```mermaid
sequenceDiagram
    actor Librarian
    participant LoanController
    participant LoanService
    participant ReaderRepository
    participant BookCopyRepository
    participant LoanRepository

    Librarian->>+LoanController: issueCopy(readerId, copyId)
    LoanController->>+LoanService: createLoan(readerId, copyId)

    LoanService->>+ReaderRepository: findById(readerId)
    ReaderRepository-->>-LoanService: Reader(reader_id, full_name)

    LoanService->>+BookCopyRepository: findById(copyId)
    BookCopyRepository-->>-LoanService: BookCopy(copy_id, status="available")

    Note over LoanService: Перевірка: reader != null AND copy.status == "available"

    LoanService->>LoanService: loaned_at = currentDate() (поточна дата)
    LoanService->>LoanService: calculateDueDate(loaned_at, DEFAULT_LOAN_DAYS=14)

    Note over LoanService: Транзакція: збереження видачі та оновлення статусу примірника
    LoanService->>+LoanRepository: save(loan)
    LoanRepository-->>-LoanService: Loan(loan_id, reader_id, copy_id, loaned_at, due_at, returned_at=null)

    LoanService->>+BookCopyRepository: updateStatus(copyId, "on_loan")
    BookCopyRepository-->>-LoanService: BookCopy(copy_id, status="on_loan")

    LoanService-->>LoanController: LoanCreatedResult(loan_id, due_at)
    LoanController-->>Librarian: LoanCreatedResult(loan_id, due_at)

    deactivate LoanService
    deactivate LoanController
```

## S-03: Відмова у видачі зайнятого примірника

```mermaid
sequenceDiagram
    actor Librarian
    participant LoanController
    participant LoanService
    participant ReaderRepository
    participant BookCopyRepository

    Librarian->>+LoanController: issueCopy(readerId, copyId)
    LoanController->>+LoanService: createLoan(readerId, copyId)

    LoanService->>+ReaderRepository: findById(readerId)
    ReaderRepository-->>-LoanService: Reader(reader_id, full_name)

    LoanService->>+BookCopyRepository: findById(copyId)
    BookCopyRepository-->>-LoanService: BookCopy(copy_id, status="on_loan")

    alt copy.status == "on_loan"
        LoanService-->>LoanController: CopyUnavailableError("Примірник недоступний")
        LoanController-->>Librarian: CopyUnavailableError("Примірник недоступний")
    else copy.status == "available"
        Note over LoanService: Продовження оформлення видачі (див. S-02)
    end

    deactivate LoanService
    deactivate LoanController
```

## S-07: Відмова у видачі незареєстрованому читачеві

```mermaid
sequenceDiagram
    actor Librarian
    participant LoanController
    participant LoanService
    participant ReaderRepository

    Librarian->>+LoanController: issueCopy(readerId, copyId)
    LoanController->>+LoanService: createLoan(readerId, copyId)

    LoanService->>+ReaderRepository: findById(readerId)
    ReaderRepository-->>-LoanService: null

    alt reader == null
        LoanService-->>LoanController: ReaderNotFoundError("Читача не зареєстровано")
        LoanController-->>Librarian: ReaderNotFoundError("Читача не зареєстровано")
    else reader != null
        Note over LoanService: Перевірка примірника та створення видачі (див. S-02)
    end

    deactivate LoanService
    deactivate LoanController
```
