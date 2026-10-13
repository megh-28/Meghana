# AI INTERACTION LOG
### Interaction Log 1: Activity 1 — Initial LibraryService Generation

| Field | What goes in it |
|---|---|
| **Tool** | GitHub Copilot in IntelliJ IDEA *(confirm this matches the assistant you actually used)* |
| **Prompt** | ```java
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
| **Decision** | **Accepted** as the *raw AI output* for the required first Git checkpoint, **not** as the final SOLID-compliant design. *(Only keep this if it reflects what you actually did.)* |
| **Fix** | No change to the raw code before its first commit. The planned Activity 3 improvement is to extract persistence behind repository interfaces and inject them into a refactored service. This is a plan, not a completed fix. |
| **Learned** | AI-generated code can cover the required functionality while still mixing business logic and persistence. Audit actual dependencies before claiming a SOLID violation. |
