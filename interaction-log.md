# Lab 7 — AI Interaction Log

## Scope and provenance

- **Seven fields used for every significant interaction:** Tool, Prompt, Output, Issues, Decision, Fix, Learned.
- **Environment:** IntelliJ IDEA, Java 21, Maven. JUnit 5 smoke-test file is present; a passing test run has not been verified.
- **Prompt drafting/documentation tool:** ChatGPT in browser. **Generation tool:** replace `[confirm Copilot/ChatGPT and model]` in the entries below with the assistant actually used in IntelliJ/browser; source archives do not establish model identity.
- **Evidence reviewed:** `lab7(3).zip` Java files. Detailed checks and file/line references: [`pattern_audit.md`](pattern_audit.md).
- **Raw transcripts:** Not supplied. If GitHub Copilot allows exporting the conversation, save it under `lab7/transcripts/` and add the actual relative link for each entry. Do **not** claim a transcript exists until exported.
- **Source preservation (user-confirmed):** The Java files were not changed after the four prompts. All generated code was accepted as produced, including any comments referring to design flaws. No manual fixes or refactoring were performed during Activities 1–2. The log reports observed defects without claiming they were deliberately inserted by the user.

## Activity 1 — Generation interactions

### Interaction 1 — Baseline — independent generation

| Field | Record |
|---|---|
| **Tool** | `[confirm Copilot in IntelliJ / ChatGPT in browser; model if known]` — prompts were drafted with ChatGPT. |
| **Prompt** | Exact text below (provided in this conversation). Transcript: `[add actual export link if available]`. |
| **Output** | 10 Java source files in `baseline/`: `Discount`, `FixedAmountDiscount`, `Main`, `Order`, `OrderItem`, `OrderNotifier`, `OrderProcessor`, `OrderStatus`, `PaymentProcessor`, `PercentageDiscount`. |
| **Issues** | Initial inspection: the pattern-neutral baseline nevertheless uses a `Discount` interface (`baseline/Discount.java`) and injected dependencies (`baseline/OrderProcessor.java:7–12`). This is an observation, not a failure; compare the five-step implementation at `baseline/OrderProcessor.java:15–27`. |
| **Decision** | **Accepted unchanged.** The user confirms no source code was changed after this generation prompt. |
| **Fix** | None in Activity 1. Keep the original baseline unchanged. |
| **Learned** | A pattern-neutral prompt may still yield abstractions; verify what the AI actually produced. |

**Exact prompt**

```text
Generate a Java 21 e-commerce order-processing application.
The system should support percentage and fixed-amount discounts, order notifications and an order-processing pipeline consisting of validation, total calculation, discount application, payment processing and notification.
Include an Order model and a runnable Main class.
Choose the implementation structure yourself. Keep it simple and self-contained using standard Java libraries.
Save the generated code in the baseline package. Do not modify existing implementations or tests.
```

### Interaction 2 — Strategy — discount implementation

| Field | Record |
|---|---|
| **Tool** | `[confirm Copilot in IntelliJ / ChatGPT in browser; model if known]` — prompts were drafted with ChatGPT. |
| **Prompt** | Exact text below (provided in this conversation). Transcript: `[add actual export link if available]`. |
| **Output** | 10 Java source files in `strategy/`, including `DiscountStrategy`, two concrete discount classes and `OrderService`. |
| **Issues** | **OCP / Strategy contract:** `DiscountStrategy` exists (`strategy/DiscountStrategy.java:5–7`), but the concrete discounts do not implement it (`PercentageDiscount.java:5`; `FixedDiscount.java:5`), and `OrderService.java:13–19` selects them with `if/else`. An interface declaration alone does not provide polymorphism. `strategy/OrderNotifier.java:5` also references an unavailable `getCustomerEmail()` method. |
| **Decision** | **Accepted unchanged.** The user confirms no source code was changed after this generation prompt. |
| **Fix** | None in Activity 1. Leave the type selection and disconnected interface for the subsequent refactor. |
| **Learned** | Specifying an interface does not guarantee that the client and variants use it. |

**Exact prompt**

```text
Generate a Java 21 e-commerce order-processing system using the Strategy design pattern for discounts.
Define a DiscountStrategy interface with a single apply(Order) method. Support percentage-based and fixed-amount discounts.
Include an OrderService responsible for processing orders, applying discounts and coordinating notifications.
You may choose how strategies are created, selected and passed to OrderService. Include a runnable Main class demonstrating multiple discount options.
Save the code in the strategy package. Generate the implementation without reviewing or refactoring it.
```

### Interaction 3 — Observer — notification implementation

| Field | Record |
|---|---|
| **Tool** | `[confirm Copilot in IntelliJ / ChatGPT in browser; model if known]` — prompts were drafted with ChatGPT. |
| **Prompt** | Exact text below (provided in this conversation). Transcript: `[add actual export link if available]`. |
| **Output** | 12 Java source files in `observer/`, including `OrderObserver`, `OrderService`, `EmailNotifier` and `LogNotifier`. |
| **Issues** | **DIP / coupled observer:** The subject has an interface list and subscription (`observer/OrderService.java:11–12,21–23`), but constructs `EmailNotifier` and `LogNotifier` directly (`observer/OrderService.java:14–19`). Older discount/payment files also call methods absent from the current `observer/Order.java` (see `pattern_audit.md`). |
| **Decision** | **Accepted unchanged.** The user confirms no source code was changed after this generation prompt. |
| **Fix** | None in Activity 1. Leave concrete observer construction unchanged until after the audit commit. |
| **Learned** | A subject can have `subscribe(OrderObserver)` yet still violate DIP by creating concrete observers. |

**Exact prompt**

```text
Generate a Java 21 e-commerce order-processing system using the Observer design pattern for notifications.
Define an OrderObserver interface and provide a subscribe(OrderObserver) method in the order-processing subject.
The system must send email and logging notifications when an order is successfully placed.
You may decide how observers are created, stored and notified. Include order discounts, payment processing and a runnable Main class.
Save the code in the observer package. Generate the initial implementation without reviewing or refactoring it.
```

### Interaction 4 — Template Method — order pipeline

| Field | Record |
|---|---|
| **Tool** | `[confirm Copilot in IntelliJ / ChatGPT in browser; model if known]` — prompts were drafted with ChatGPT. |
| **Prompt** | Exact text below (provided in this conversation). Transcript: `[add actual export link if available]`. |
| **Output** | 7 Java source files in `template/`, including `OrderProcessor`, `StandardOrderProcessor` and `PremiumOrderProcessor`. |
| **Issues** | No structural pattern violation found in the inspected source: `processOrder()` is final and invokes five ordered steps (`template/OrderProcessor.java:8–15`), while subclasses override hooks (`StandardOrderProcessor.java:5–14`; `PremiumOrderProcessor.java:5–15`). Still requires JUnit seam tests in Activity 3. |
| **Decision** | **Accepted unchanged.** The user confirms no source code was changed after this generation prompt. |
| **Fix** | None in Activity 1. Keep the final pipeline and subclasses unchanged. |
| **Learned** | The Template Method seam is the final skeleton plus overridable hooks, not just five sequential calls. |

**Exact prompt**

```text
Generate a Java 21 e-commerce order-processing system using the Template Method design pattern.
Create an abstract OrderProcessor with a processOrder() method that executes five steps in order: validateOrder, calculateTotal, applyDiscount, processPayment and sendNotification.
Implement StandardOrderProcessor and PremiumOrderProcessor with different discount or notification behavior.
You may choose method visibility, inheritance structure and how individual processing steps are implemented.
Include an Order model and runnable Main class. Save the code in the template package. Do not perform a design audit or refactoring during generation.
```

## Activity 2 — AI-assisted pattern audit

| Field | Record |
|---|---|
| **Tool** | ChatGPT in browser (audit documentation assistance). |
| **Prompt** | “Create `lab7/pattern_audit.md` with one section per pattern; cite source lines; check Strategy, Observer and Template Method; classify failed checks and conclude with a justified verdict; do not edit code before the audit commit.” (Faithful summary of the full audit instructions supplied in this chat.) |
| **Output** | [`pattern_audit.md`](pattern_audit.md): line-cited checks and verdicts — Strategy misapplied (interface not used), Observer misapplied (coupled observer), Template Method correct. |
| **Issues** | The archive contains inconsistent Java APIs and comments labeling certain flaws. Per user confirmation, these are part of the unchanged generated code; the audit separately identifies design violations and compile-time API mismatches. |
| **Decision** | **Accepted as documentation; no source code was changed.** Verify that the audited source matches the repository commit. |
| **Fix** | No Java changes. Documented separate compilation inconsistencies in the audit; made no Java changes. |
| **Learned** | A pattern can fail despite having the expected interface names; verify actual dependencies and calls. |

## Activity 3 — Future interactions (not yet performed)

For each Copilot/ChatGPT generation, check, refactor, test or rejection, add a new seven-field entry using this template:

| Field | Record |
|---|---|
| **Tool** | Exact tool, location and model. |
| **Prompt** | Exact prompt or summary plus an actual transcript link. |
| **Output** | What code/test/review was produced. |
| **Issues** | Observed defect or risk, with file/line where possible. |
| **Decision** | Accepted / modified / rejected. |
| **Fix** | What *actually* changed and why; `None` if unchanged. |
| **Learned** | One transferable lesson. |

## Git and verification record

- Raw-generation commit: `[paste actual hash]`
- Pattern-audit commit: `[paste actual hash]`
- Smoke-test outcome: `[paste command and actual result]`
- Full-suite outcome after Activity 3: `[paste command and actual result]`
- Transcript files/links: `[only add after export]`

> The user confirms the four generated implementations were preserved unchanged after their prompts. Keep the raw-generation commit and audit commit as separate Git checkpoints; log any future changes as new interactions.
