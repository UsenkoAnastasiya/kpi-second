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

    opt reader != null
        LoanService->>+BookCopyRepository: findById(copyId)
        BookCopyRepository-->>-LoanService: BookCopy(copy_id, status="available")

        opt copy.status == "available"
            LoanService->>LoanService: calculateDueDate(loaned_at, DEFAULT_LOAN_DAYS=14)
            LoanService->>+LoanRepository: save(loan)
            LoanRepository-->>-LoanService: Loan(loan_id, reader_id, copy_id, loaned_at, due_at, returned_at=null)

            LoanService->>+BookCopyRepository: updateStatus(copyId, "on_loan")
            BookCopyRepository-->>-LoanService: BookCopy(copy_id, status="on_loan")
        end
    end

    LoanService-->>-LoanController: LoanCreatedResult(loan_id, due_at)
    LoanController-->>-Librarian: 201 Created (loan_id, due_at)
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
        LoanController-->>Librarian: 409 Conflict ("Примірник недоступний")
    else copy.status == "available"
        Note over LoanService: Продовження оформлення видачі (див. S-02)
        LoanService-->>LoanController: LoanCreatedResult(loan_id, due_at)
        LoanController-->>Librarian: 201 Created (loan_id, due_at)
    end

    deactivate LoanService
    deactivate LoanController
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
        LoanController-->>Librarian: 404 Not Found ("Читача не зареєстровано")
    else reader != null
        Note over LoanService: Перевірка примірника та створення видачі (див. S-02)
        LoanService-->>LoanController: LoanCreatedResult(loan_id, due_at)
        LoanController-->>Librarian: 201 Created (loan_id, due_at)
    end

    deactivate LoanService
    deactivate LoanController
```
