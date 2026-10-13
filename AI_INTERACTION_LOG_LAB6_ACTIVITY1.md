# AI INTERACTION LOG
## Lab 6 — The SOLID Audit: Reviewing an AI-Generated Service

### Interaction Log 1: Activity 1 — Initial LibraryService Generation

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA *(confirm this matches the assistant you actually used)* |
| **Prompt** | 
```java
// LibraryService manages borrowing and returning books for a library system.
// It maintains a list of books and members. Key methods:
// borrowBook(int memberId, String isbn) — marks book as borrowed
// returnBook(int memberId, String isbn) — marks book as available
// searchBooks(String query)           — returns matching books
// notifyOverdueMembers()               — emails members with overdue books
// Include persistence: save and load books/members from a MySQL database.
``` |
| **Output** | Generated `LibraryService.java` with book/member registration, borrowing, returning, searching, overdue detection/notification, and JDBC save/load/schema operations. The shared files also include a focused `OverdueEmailSender` interface. |
| **Issues** | **SRP:** The same service implements library business rules and SQL persistence (`saveToDatabase`, `loadFromDatabase`, `createSchema`). **DIP:** Business-service code imports and directly uses JDBC (`Connection`, `PreparedStatement`, `ResultSet`, `Statement`) instead of relying on repository interfaces. However, the service does **not** create its own database connection or concrete email sender: the connection is passed into methods and the `OverdueEmailSender` interface is injected. No clear LSP or ISP violation was found in the shared files. |
| **Decision** | **Accepted** as the *raw AI output* for the required first Git checkpoint, **not** as the final SOLID-compliant design.|
| **Fix** | **Fix:** Refactor `LibraryService` to separate business logic from database persistence. Create `BookRepository` and `MemberRepository` interfaces to handle data access and move SQL queries, database connections, and schema creation into separate concrete repository implementations. Inject these repositories through the `LibraryService` constructor instead of handling JDBC operations directly inside the service. Retain the existing `OverdueEmailSender` abstraction for notifications. This addresses the SRP and DIP violations and allows the business logic to be tested independently using JUnit 5 and Mockito. |
| **Learned** | AI-generated code can cover the required functionality while still mixing business logic and persistence. Audit actual dependencies before claiming a SOLID violation. |
