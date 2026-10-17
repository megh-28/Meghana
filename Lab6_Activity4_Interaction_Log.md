# AI INTERACTION LOG
### Interaction Log 1: Activity 1 — Initial LibraryService Generation

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA |
| **Prompt** | ```java // LibraryService manages borrowing and returning books for a library system.// It maintains a list of books and members. Key methods: // borrowBook(int memberId, String isbn) — marks book as borrowed // returnBook(int memberId, String isbn) — marks book as available // searchBooks(String query)           — returns matching books // notifyOverdueMembers()— emails members with overdue books // Include persistence: save and load books/members from a MySQL database.``` |
| **Output** | Generated `LibraryService.java` with book/member registration, borrowing, returning, searching, overdue detection/notification, and JDBC save/load/schema operations. The shared files also include a focused `OverdueEmailSender` interface. |
| **Issues** | **SRP:** The same service implements library business rules and SQL persistence (`saveToDatabase`, `loadFromDatabase`, `createSchema`). **DIP:** Business-service code imports and directly uses JDBC (`Connection`, `PreparedStatement`, `ResultSet`, `Statement`) instead of relying on repository interfaces. However, the service does **not** create its own database connection or concrete email sender: the connection is passed into methods and the `OverdueEmailSender` interface is injected. No clear LSP or ISP violation was found in the shared files. |
| **Decision** | **Accepted** as the *raw AI output* for the required first Git checkpoint, **not** as the final SOLID-compliant design. |
| **Fix** |  Refactor `LibraryService` to separate business logic from database persistence. Create `BookRepository` and `MemberRepository` interfaces to handle data access and move SQL queries, database connections, and schema creation into separate concrete repository implementations. Inject these repositories through the `LibraryService` constructor instead of handling JDBC operations directly inside the service. Retain the existing `OverdueEmailSender` abstraction for notifications. This addresses the SRP and DIP violations and allows the business logic to be tested independently using JUnit 5 and Mockito. |
| **Learned** | AI-generated code can cover the required functionality while still mixing business logic and persistence. Audit actual dependencies before claiming a SOLID violation. |

### Interaction Log 2: MemberRepository.java

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt** | Create a MemberRepository interface with Optional<Member> findById(int) and save(Member). |
| **Output** | MemberRepository interface for member lookup and updates. |
| **Issues** | No clear problem in the proposed interface. |
| **Decision** | Accepted as part of the revised design. |
| **Fix** | No code change required for this interface. |
| **Learned** | Separate membership lookup from the library service. |


### Interaction Log 3: ConsoleNotificationService.java

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt (summary; not verbatim)** | Create ConsoleNotificationService that prints member email and overdue title instead of connecting to SMTP. |
| **Output** | Concrete console notification implementation. |
| **Issues** | Console output is not an actual email. |
| **Decision** | Accepted as the lab stub. |
| **Fix** | Used console output to avoid infrastructure during the demonstration. |
| **Learned** | Concrete notification behaviour can be replaced behind an interface. |

### Interaction Log 4: LibraryService.java

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt (summary; not verbatim)** | Refactor LibraryService using injected BookRepository, MemberRepository, and NotificationService, keeping SQL out of business logic. |
| **Output** | Constructor-injected LibraryService with borrowing logic. |
| **Issues** | Initial version omitted returnBook, searchBooks, 14-day loan state, and overdue detection. |
| **Decision** | Modified. |
| **Fix** | Extended the service with loan abstraction and missing business operations. |
| **Learned** | Improving design must not remove required behaviour. |

### Interaction Log 5: Loan.java and loan repositories

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt (summary; not verbatim)** | Create Loan record, LoanRepository interface, and InMemoryLoanRepository to restore ISBN, member ID, and due-date storage. |
| **Output** | Loan record and loan repository abstractions with in-memory adapter. |
| **Issues** | The initial refactoring had lost loan state. |
| **Decision** | Modified. |
| **Fix** | Added LoanRepository constructor injection and 14-day due-date storage using Clock. |
| **Learned** | Domain state must survive refactoring of storage boundaries. |


### Interaction Log 6: Mockito regression tests

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt (summary; not verbatim)** | Add Mockito tests for return, wrong borrower, search, and overdue notifications, using fixed Clock and mocked dependencies. |
| **Output** | Six-test suite covering borrowing, returning, searching, and overdue notifications. |
| **Issues** | Generated tests needed execution-based verification. |
| **Decision** | Accepted after testing. |
| **Fix** | Maven execution showed 6 tests, 0 failures, 0 errors, 0 skipped; BUILD SUCCESS. |
| **Learned** | Test results become evidence only after execution. |

### Interaction Log 7:  LibraryApp verification

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA  |
| **Prompt (summary; not verbatim)** | Update the app to construct LoanRepository and demonstrate borrow and return by printing active loan counts. |
| **Output** | Runnable composition root demonstrating in-memory loan count changes. |
| **Issues** | It does not demonstrate actual MySQL or email infrastructure. |
| **Decision** | Accepted after execution. |
| **Fix** | Observed counts 1 after borrow and 0 after return; exit code 0. |
| **Learned** | Runnable integration checks complement mock-based tests. |
A
## Interaction Log 8 Activity 4— Generating a SOLID-Compliant LibraryService

| Field | Description |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA (as reported; verify against your actual session) |
| **Prompt** | Generate a SOLID-compliant implementation in the package ie.ucd.comp41670.lab6.library.cleanprompt. Support borrowing, returning, searching books by ISBN/title/author, default 14-day loans, overdue detection and notifications. Separate business logic from persistence and notification delivery using small repository and notification interfaces. Use constructor injection and final fields. Include simple in-memory concrete implementations and a main application that wires them together. Use java.time.Clock to make overdue logic testable. Do not put JDBC, SQL, SMTP, or concrete infrastructure construction inside LibraryService. Validate unknown members, unknown books, unavailable books and incorrect borrowers. Keep all existing original and refactored classes untouched.Existing domain classes are Book and Member in the package ie.ucd.comp41670.lab6.library.|
| **Output** | Generated 13 Java files in `ie.ucd.comp41670.lab6.library.cleanprompt`, including domain records, interfaces, in-memory implementations, `LibraryService`, and `LibraryApp`. |
| **Issues** | Copilot created new `Book` and `Member` records instead of reusing the original models. Book availability is derived from active loans instead of a mutable flag. The availability check and loan save are separate, presenting a possible concurrency race. |
| **Decision** | **Accepted ** Otherwise mark **Modified** and explain actual edits. |
| **Fix** | No SOLID refactor was required based on the five-point static audit. |
| **Learned** | Explicit constructor-injection and infrastructure-separation requirements helped avoid the SRP and DIP problems observed in Activity 2. |



## 1. Generated Output and Transcript

The generated `cleanprompt` package contains:

1. `Book.java`
2. `Member.java`
3. `Loan.java`
4. `BookRepository.java`
5. `MemberRepository.java`
6. `LoanRepository.java`
7. `NotificationService.java`
8. `InMemoryBookRepository.java`
9. `InMemoryMemberRepository.java`
10. `InMemoryLoanRepository.java`
11. `ConsoleNotificationService.java`
12. `LibraryService.java`
13. `LibraryApp.java`


## 2. Five-Point SOLID Mini-Audit

| Principle | Result | Evidence from generated code |
|---|---|---|
| **S — Single Responsibility (SRP)** | **PASS** | `LibraryService` implements borrowing, returning, searching, and overdue rules; repositories manage storage and `NotificationService` handles notification delivery. The service contains no SQL or SMTP implementation. |
| **O — Open/Closed (OCP)** | **PASS — no relevant violation observed** | No `switch` or type/category-based dispatch controls book, member, or notification behaviour. The service calls interfaces rather than branching over concrete infrastructure types. |
| **L — Liskov Substitution (LSP)** | **PASS — no violation demonstrated** | In-memory repositories and console notification adapter implement their declared operations; no `UnsupportedOperationException` or contract-breaking override was identified in the reviewed code. |
| **I — Interface Segregation (ISP)** | **PASS** | `BookRepository`, `MemberRepository`, `LoanRepository`, and `NotificationService` each expose focused operations; implementations are not forced to implement unrelated methods. |
| **D — Dependency Inversion (DIP)** | **PASS** | `LibraryService` has `private final` fields for repository/notification interfaces, takes these through its constructors, and does not construct concrete infrastructure. `LibraryApp` creates the in-memory implementations and injects them. |

### Constructor-injection evidence

```java
private final BookRepository bookRepository;
private final MemberRepository memberRepository;
private final LoanRepository loanRepository;
private final NotificationService notificationService;
private final Clock clock;
```

This is a *static source-code assessment* of the generated output, not proof of all possible runtime behaviours.

## 4. Issues and Limitations

**Separate domain models.** Copilot generated fresh `Book` and `Member` records inside `cleanprompt` instead of importing the original domain classes. This is a divergence from the context, although it preserves independence for the experiment.

**Different availability model.** `Book` has no mutable `available` field. The service treats an existing loan record as evidence that the book is unavailable. This is a domain-design difference, not by itself a SOLID violation.

**Concurrent borrowing risk.** `borrowBook()` checks `loanRepository.findByIsbn(...)` before calling `loanRepository.save(...)`. Since the operations are separate, two concurrent callers could both pass the availability check before either stores a loan. This potential race is not resolved by the five SOLID principles alone.

**Test scope.** The demonstrated application run is not a complete regression suite for the new `cleanprompt` implementation.

## 5. Most Influential Prompt Constraint

The most useful constraint was requiring **small repository and notification interfaces with constructor injection, and prohibiting JDBC, SQL, SMTP, and concrete infrastructure construction inside `LibraryService`**. This directly targeted the SRP and DIP issues identified in the original Activity 2 audit.