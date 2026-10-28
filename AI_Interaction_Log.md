# Lab 8 — AI Interaction Log

## Interaction 1 — Generating the initial JUnit 5 test suite

**Tool:** AI assistant used in IntelliJ IDEA (confirm the assistant name, e.g. IntelliJ AI Assistant or GitHub Copilot).

**Prompt:**

> The Java source files for my order management system are in this path:
> C:\Users\megha\IdeaProjects\comp41670-labs\lab8\src\main\java\ie\ucd\comp41670\orders
> Can you generate JUnit 5 test cases for these classes so I can check if the methods are working properly?
> Please give me the complete test files with filenames like ProductTest.java, OrderTest.java, OrderBookTest.java, and so on.
> Place the test files in the corresponding package under src/test/java. Include the necessary imports so I can run them in IntelliJ.

**Output:** The assistant generated six JUnit 5 test files: `ProductTest.java`, `OrderLineTest.java`, `OrderStatusTest.java`, `OrderTest.java`, `OrderBookTest.java`, and `PricingCalculatorTest.java`. Together they contain 27 test methods.

**Issues:** The suite compiled and executed, but five tests failed (two in `OrderTest` and three in `PricingCalculatorTest`), with 22 passing. These are assertion failures that require comparison against `SPEC.md`; they could reflect differences between the expected and implemented behaviour, rather than errors in the generated tests. High code coverage alone does not establish behavioural correctness.

**Decision:** **Accepted** as the raw AI-generated suite for Activity 1, retaining the five failing tests for subsequent examination.

**Fix:** No changes to assertions or production code are documented here. **Before submission, confirm whether any compilation fixes were made; if so, list each exact fix and why it was needed.** The JaCoCo report was generated separately using `./gradlew.bat jacocoTestReport -x test` after the normal test task failed.

**Learned:** AI-generated tests can reach high code coverage while still needing careful examination of their expectations and assertions.

