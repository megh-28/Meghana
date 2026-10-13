# Lab 6 — The SOLID Audit

## 1. Graduated review: observations before applying SOLID labels

**Class and fields.** `LibraryService` owns three in-memory maps: `Map<String, Book> books`, `Map<Integer, Member> members`, and `Map<String, Loan> loans`. It also stores the loan period, a `Clock`, and an `OverdueEmailSender`. Its constructors accept an email-sender *interface*, rather than constructing a concrete email sender.

**Signatures and imports.** Borrow/return/search/overdue operations sit alongside `saveToDatabase(Connection connection)` and `loadFromDatabase(Connection connection)`. Infrastructure-specific imports include `java.sql.Connection`, `PreparedStatement`, `ResultSet`, `SQLException`, `Statement`, and `Timestamp`.

**Bodies.** The service performs borrowing rules and overdue grouping, but also issues SQL `DELETE`, `INSERT`, `SELECT`, and `CREATE TABLE` statements; manages JDBC statements and transactions; and reconstructs its domain objects from `ResultSet` rows.

## 2. Five-point SOLID checklist

| Principle | Pass / Fail | Quoted evidence from raw code | Proposed fix / reason |
|---|---|---|---|
| **S — Single Responsibility (SRP): multi-concern?** | **FAIL** | `public synchronized void borrowBook(int memberId, String isbn)` appears in the same class as `public synchronized void saveToDatabase(Connection connection) throws SQLException` and `private static void createSchema(Connection connection) throws SQLException`. The latter executes `statement.executeUpdate("DELETE FROM library_loans");` and SQL table definitions. | Move JDBC persistence/schema operations into dedicated repository/infrastructure classes. Keep borrowing, returning, searching, and overdue policy in the service. |
| **O — Open/Closed (OCP): type-switch?** | **PASS — no type-switch identified** | `searchBooks` uses `.filter(book -> contains(book.getTitle(), normalizedQuery) || contains(book.getAuthor(), normalizedQuery) || contains(book.getIsbn(), normalizedQuery))`; the borrowing methods check availability and loan ownership, not a book/member type discriminator. | No type-based conditional dispatch requiring changes for new subtypes was found in this raw code. Reassess if categories or notification channels are introduced. |
| **L — Liskov Substitution (LSP): throwing overrides?** | **PASS — no violation demonstrated** | The supplied model and service classes do not declare inheritance-based overrides; `public interface OverdueEmailSender { void sendOverdueNotice(Member member, List<Book> overdueBooks); }` provides a contract, but no contract-breaking implementation appears in the reviewed files. | Do not invent an LSP violation. Recheck if subclasses or implementations are later added. |
| **I — Interface Segregation (ISP): empty implementations?** | **PASS — no violation demonstrated** | `OverdueEmailSender` has only `void sendOverdueNotice(Member member, List<Book> overdueBooks);`. No empty or irrelevant interface implementations appear in the reviewed files. | The notification contract is focused. Keep future persistence and notification interfaces small and role-specific. |
| **D — Dependency Inversion (DIP): hardcoded infrastructure?** | **FAIL — direct JDBC coupling; qualified finding** | `import java.sql.Connection;`, `public synchronized void loadFromDatabase(Connection connection) throws SQLException`, and `statement.executeQuery("SELECT isbn, title, author, available FROM library_books")` show that `LibraryService` itself knows JDBC and SQL. | Introduce interfaces such as `BookRepository` and `MemberRepository` (and a loan repository or equivalent as needed), inject them via constructor, and keep SQL inside concrete adapters. **Qualification:** the raw service does **not** use `new` to create its own `Connection`, DAO, or email sender. It already constructor-injects `OverdueEmailSender`; its DIP problem is the direct JDBC API/SQL dependency in high-level service code. |

## 3. Two evidence-based findings

### Finding 1 — Most critical: SRP violation from embedded persistence

`LibraryService` declares both `borrowBook(int memberId, String isbn)` and `saveToDatabase(Connection connection)`, while `createSchema()` directly executes SQL such as `CREATE TABLE IF NOT EXISTS library_books`. This gives a single service two unrelated reasons to change—library borrowing rules and database schema/persistence—and means even a persistence-only change requires editing business-service code; the SQL-heavy save/load paths also require JDBC infrastructure or more elaborate fakes to test.

### Finding 2 — DIP violation, despite one good injection choice

The service directly accepts `Connection` and executes queries such as `SELECT isbn, title, author, available FROM library_books`, so the high-level service is tied to JDBC and a particular relational schema instead of repository interfaces. However, `LibraryService(OverdueEmailSender emailSender)` demonstrates a *successful* abstraction boundary for notifications: the refactor should preserve that pattern and apply it to persistence, not falsely claim the email sender is instantiated with `new`.

## 4. Audit scope / evidence limits

This is a **static source-code audit**, not evidence that a real MySQL save/load operation or SMTP email delivery has been run. The audit intentionally marks OCP/LSP/ISP as passing **only for the supplied code**, not as a promise that every future extension will preserve those principles. The DIP checklist's specific `hardcoded new` smell is *absent for infrastructure*, even though a broader DIP violation exists because JDBC and SQL remain in the service.
