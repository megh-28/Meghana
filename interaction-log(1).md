# AI Interaction Log

## Interaction Log 1

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Field                               What goes in it
  ----------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Tool**                            GitHub Copilot

  **Prompt**                          `// Context: class for a library lending system, used by services that must never see an invalid book.``<br>`{=html}`<br>`{=html}`// Intent: LibraryBookAI with private title/author/isbn/available; constructor validates non-null,``<br>`{=html}`// non-blank args (IllegalArgumentException); checkOut()/returnBook() guard state (IllegalStateException);``<br>`{=html}`// getters only, no setters for immutable fields.``<br>`{=html}`// Evaluation: list any edge cases this implementation does not handle`

  **Output**                          GitHub Copilot generated the `LibraryBookAI` class with private `title`, `author`, `isbn`, and `available` fields. The constructor validates that title, author, and ISBN are not null or blank. The `checkOut()` and `returnBook()` methods validate the current availability of the book and throw `IllegalStateException` for invalid operations (second checkout, second return). Getters are provided for the immutable fields (using `final`), with no setters. However, it listed edge cases such as
                                      `checkOut()` and `returnBook()` not being concurrency-safe: two threads could both check out the same book or return it twice.

  **Issues**                          The implementation handles null and ordinary blank strings, but some edge cases are not fully handled. The class does not prevent two threads from simultaneously checking out or returning the same book because the state-changing methods are not synchronized.

  **Decision**                        The raw AI output was reviewed rather than blindly accepted. The basic validation is suitable, but additional edge cases should be considered, such as two threads simultaneously checking out or returning the same book.

  **Fix**                             No immediate fix was made because the prompt did not explicitly require thread safety.

  **Learned**                         AI-generated code can satisfy the main requirements while still missing important edge cases.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Question Review

### Q1 --- Does it run?

Yes. The raw AI version of the `LibraryBookAI` class was compiled, and
10 test cases passed.

However, the concurrency tests showed a problem when two `checkOut()`
operations, two `returnBook()` operations, or a `checkOut()` and
`returnBook()` happened at the same time. The normal boolean state was
not updated atomically, so two operations could succeed when only one
should have succeeded. This showed that the use of `compareAndSet()`
with `AtomicBoolean` was necessary to make the state changes atomic.

### Q2 --- Does it do what I asked?

**Mostly yes.** The constructor validates null and blank values, the
checkout and return methods guard the book state, and the immutable
fields have getters without setters. However, the code uses a normal
boolean for availability, so it does not safely handle simultaneous
`checkOut()` and `returnBook()` operations.

### Q3 --- Did Copilot add fields, setters or methods you did not ask for?

It added an extra method named `validateRequiredText()` for automatic
whitespace normalization that was not explicitly asked for.

### Q4 --- Does it hold each of the written invariants?

**Partially.** The constructor validation, state guards, and absence of
setters satisfy the main written invariants. However, the use of a
normal boolean availability state is not safe if `checkOut()` and
`returnBook()` happen concurrently.

### Q5 --- Could I defend every line to a colleague?

**Mostly yes.** I can explain the constructor, validation method,
`checkOut()`, `returnBook()`, getters, and availability logic. However,
I identified that a normal boolean does not make simultaneous state
changes safe. These issues needed to be addressed through a targeted
re-prompt.

## Final Interaction Log

  -----------------------------------------------------------------------
  Field                               What goes in it
  ----------------------------------- -----------------------------------
  **Tool**                            GitHub Copilot in IntelliJ IDEA

  **Prompt**                          Modify the code such that it can
                                      handle `checkOut()` and
                                      `returnBook()` if they happen at
                                      the same time.

  **Output**                          Copilot changed `available` from
                                      `boolean` to `AtomicBoolean` and
                                      used `compareAndSet()` in
                                      `checkOut()` and `returnBook()`.

  **Issues**                          The original implementation used a
                                      normal boolean, so the
                                      check-then-update operation was not
                                      atomic and could have a race
                                      condition when multiple threads
                                      accessed the book simultaneously.

  **Decision**                        Accepted the corrected
                                      implementation.

  **Fix**                             Changed `available` to
                                      `AtomicBoolean`; changed
                                      `checkOut()` to use
                                      `compareAndSet(true, false)` and
                                      `returnBook()` to use
                                      `compareAndSet(false, true)`.

  **Learned**                         A class can satisfy its basic
                                      functional specification while
                                      still having concurrency edge
                                      cases. Targeted re-prompts are
                                      useful because they address
                                      specific problems identified during
                                      human review rather than blindly
                                      accepting new AI-generated code.
  -----------------------------------------------------------------------
