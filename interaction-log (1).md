# Lab 7 — AI Interaction Log

### Interaction 1 — Baseline — independent generation

| Field | Record |
|---|---|
| **Tool** | ` IntelliJ Copilot`  |
| **Prompt** | ```text Generate a  e-commerce order-processing application.The system should support percentage and fixed-amount discounts, order notifications and an order-processing pipeline consisting of validation, total calculation, discount application, payment processing and notification.Include an Order model and a runnable Main class.Choose the implementation structure yourself.Keep it simple and self-contained using standard Java libraries.Save the generated code in the baseline package.``` |
| **Output** | 10 Java source files in `baseline/`: `Discount`, `FixedAmountDiscount`, `Main`, `Order`, `OrderItem`, `OrderNotifier`, `OrderProcessor`, `OrderStatus`, `PaymentProcessor`, `PercentageDiscount`. |
| **Issues** | Initial inspection: the pattern-neutral baseline nevertheless uses a `Discount` interface (`baseline/Discount.java`) and injected dependencies (`baseline/OrderProcessor.java`). This is an observation, not a failure|
| **Decision** | **Accepted unchanged.**  |
| **Fix** |  No design fix is needed . In Activity 3, will verify that the baseline still preserves the five-step order and expected totals/notifications; only repair confirmed behavioural defects.|
| **Learned** | A pattern-neutral prompt may still yield abstractions; verify what the AI actually produced. |

### Interaction 2 — Strategy — discount implementation

| Field | Record |
|---|---|
| **Tool** | `IntelliJ Copilot`  |
| **Prompt** | ```text Generate a e-commerce order-processing system using the Strategy design pattern for discounts.Define a DiscountStrategy interface with a single apply(Order) method. Support percentage-based and fixed-amount discounts. Include an OrderService responsible for processing orders, applying discounts and coordinating notifications.You may choose how strategies are created, selected and passed to OrderService. Include a runnable Main class demonstrating multiple discount options.Save the code in the strategy package. Generate the implementation without reviewing or refactoring it.``` |
| **Output** | 10 Java source files in `strategy/`, including `DiscountStrategy`, two concrete discount classes and `OrderService`. |
| **Issues** | **OCP / Strategy contract:** `DiscountStrategy` exists (`strategy/DiscountStrategy.java`), but the concrete discounts do not implement it (`PercentageDiscount.java:5`; `FixedDiscount.java`), and `OrderService.java` selects them with `if/else`. An interface declaration alone does not provide polymorphism. `strategy/OrderNotifier.java:5` also references an unavailable `getCustomerEmail()` method. |
| **Decision** | **Accepted unchanged.**|
| **Fix** |  Make `PercentageDiscount` and `FixedDiscount` implement `DiscountStrategy` with `apply(Order)`. Inject/store a `DiscountStrategy` in `OrderService` and delegate discount computation to it; remove the discount-type `if/else`. Construct the chosen implementation outside `OrderService` (e.g., in `Main`). Resolve the `getCustomerEmail()` API mismatch by aligning `OrderNotifier` with the actual `Order` model without changing intended notification behaviour. Add a `FlashSaleDiscount` injection test to prove extensibility. |
| **Learned** | Specifying an interface does not guarantee that the client and variants use it. |


### Interaction 3 — Observer — notification implementation

| Field | Record |
|---|---|
| **Tool** | `IntelliJ Copilot`  |
| **Prompt** | ```text Generate a  e-commerce order-processing system using the Observer design pattern for notifications.Define an OrderObserver interface and provide a subscribe(OrderObserver) method in the order-processing subject.The system must send email and logging notifications when an order is successfully placed.You may decide how observers are created, stored and notified. Include order discounts, payment processing and a runnable Main class.Save the code in the observer package. Generate the initial implementation without reviewing or refactoring it.``` |
| **Output** | 12 source files in `observer/`, including `OrderObserver`, `OrderService`, `EmailNotifier` and `LogNotifier`. |
| **Issues** | **DIP / coupled observer:** The subject has an interface list and subscription (`observer/OrderService.java`), but constructs `EmailNotifier` and `LogNotifier` directly (`observer/OrderService.java`). Older discount/payment files also call methods absent from the current `observer/Order.java` (see `pattern_audit.md`). |
| **Decision** | **Accepted unchanged.** |
| **Fix** | Remove `new EmailNotifier()` and `new LogNotifier()` from the `OrderService` constructor. Instantiate and register them in `Main` through `subscribe(OrderObserver)` while retaining the `List<OrderObserver>` and notification loop; ensure each observer receives exactly one event per successful order. Align older discount/payment class method calls with the current `Order` API while preserving totals, notification messages and pipeline order. Add a hand-written fake observer seam test. |
| **Learned** | A subject can have `subscribe(OrderObserver)` yet still violate DIP by creating concrete observers. |


### Interaction 4 — Template Method — order pipeline

| Field | Record |
|---|---|
| **Tool** | `IntelliJ Copilot`|
| **Prompt** | ```text Generate a e-commerce order-processing system using the Template Method design pattern.Create an abstract OrderProcessor with a processOrder() method that executes five steps in order: validateOrder, calculateTotal, applyDiscount, processPayment and sendNotification.Implement StandardOrderProcessor and PremiumOrderProcessor with different discount or notification behavior.You may choose method visibility, inheritance structure and how individual processing steps are implemented.Include an Order model and runnable Main class. Save the code in the template package. Do not perform a design audit or refactoring during generation.``` |
| **Output** | 7  source files in `template/`, including `OrderProcessor`, `StandardOrderProcessor` and `PremiumOrderProcessor`. |
| **Issues** | No structural pattern violation found in the inspected source: `processOrder()` is final and invokes five ordered steps (`template/OrderProcessor.java`), while subclasses override hooks (`StandardOrderProcessor.java`; `PremiumOrderProcessor.java`). Still requires JUnit seam tests in Activity 3. |
| **Decision** | **Accepted unchanged.** |
| **Fix** | Retain the final `processOrder()` and protected/abstract hooks. In Activity 3, add a step-recording subclass test to verify the five calls occur in the required order; change production code only if that test reveals a defect. |
| **Learned** | The Template Method seam is the final skeleton plus overridable hooks, not just five sequential calls. |




