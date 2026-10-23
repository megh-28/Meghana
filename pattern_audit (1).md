# Lab 7 — Activity 2: Pattern Audit

**Evidence source:** `lab7(3).zip`, `lab7/src/main/java/ie/ucd/comp41670/lab7/`. All citations below are source paths and **1-based line numbers** in this uploaded ZIP. This is a read-only audit: no Java files were changed.

**Misapplication vocabulary:** (1) ceremony without need; (2) missing interface; (3) coupled observer; (4) identity swap. A failed check is classified under the closest applicable mode; unrelated compile errors are reported separately.

## 1. Strategy — discounts

| Check | Result | Source evidence |
|---|---|---|
| Shared `DiscountStrategy` interface exists? | **Pass (declaration only)** | `strategy/DiscountStrategy.java:5–7` defines `DiscountStrategy` with `BigDecimal apply(Order)`. |
| Concrete discounts implement it? | **Fail — missing interface *in use*** | `strategy/PercentageDiscount.java:5–9` and `strategy/FixedDiscount.java:5–10` declare standalone classes, without `implements DiscountStrategy`. This is a disconnected interface, rather than an absent declaration. |
| `OrderService` holds only an interface-typed strategy reference? | **Fail — missing interface in use** | `strategy/OrderService.java:6–19` contains no `DiscountStrategy` field or parameter; instead `placeOrder(Order, String)` constructs concrete discount classes. |
| Fourth discount without editing `OrderService`? | **Fail — missing interface in use** | `strategy/OrderService.java:8,13–19` selects by a closed set of string values; a new named discount needs another case or equivalent change here. |
| Any `if/switch` on discount *type*? | **Fail — missing interface in use** | `strategy/OrderService.java:13–19` branches on `"PERCENTAGE"` and `"FIXED"`. `strategy/Main.java:11,15` passes those string selectors. |

**Governing question — what varies, and who varies it?** The discount calculation varies: the percentage calculation is in `strategy/PercentageDiscount.java:6–9`, and the fixed amount calculation in `strategy/FixedDiscount.java:6–10`. The caller selects which algorithm using a string (`strategy/Main.java:7–15`), but `OrderService` hardcodes how the choice maps to classes (`strategy/OrderService.java:13–19`). Thus Strategy is warranted in principle, but the intended extension seam is blocked.

**Mode classification:** **Missing interface (in effective use)**. The interface declaration exists (`strategy/DiscountStrategy.java:5–7`), but neither concrete discount implements it (`strategy/PercentageDiscount.java:5`; `strategy/FixedDiscount.java:5`) and the context uses a type switch instead (`strategy/OrderService.java:13–19`). This is **not** an identity swap or a ceremony-only problem: two different calculations exist (`strategy/PercentageDiscount.java:7–8`; `strategy/FixedDiscount.java:7–9`).

**Verdict: MISAPPLIED — missing interface in use / concrete type-selection.** (`strategy/DiscountStrategy.java:5–7`; `strategy/OrderService.java:8–19`.)

## 2. Observer — notifications

| Check | Result | Source evidence |
|---|---|---|
| `OrderObserver` interface exists? | **Pass** | `observer/OrderObserver.java:3–5` declares `update(String, Order)`. |
| Subject stores `List<OrderObserver>`? | **Pass** | `observer/OrderService.java:11–12`. |
| `subscribe()` accepts the interface? | **Pass** | `observer/OrderService.java:21–23`. |
| Subject notifies through the list? | **Pass** | `observer/OrderService.java:57–60` iterates through `OrderObserver` objects and invokes `update`. |
| Subject avoids constructing/referencing concrete observers? | **Fail — coupled observer** | `observer/OrderService.java:14–19` directly constructs `new EmailNotifier()` and `new LogNotifier()` inside its constructor. The notifiers implement `OrderObserver` (`observer/EmailNotifier.java:4`; `observer/LogNotifier.java:3`). |

**Governing question — what varies, and who varies it?** Notification actions vary: email and logging have distinct `update` implementations (`observer/EmailNotifier.java:4–11`; `observer/LogNotifier.java:3–10`). Consumers can register a new observer through `subscribe` (`observer/OrderService.java:21–23`), so Observer has a meaningful extension seam. However, construction of the default subscribers is controlled by the subject itself (`observer/OrderService.java:14–19`), unnecessarily coupling it to particular implementations. It is therefore not merely ceremony.

**Mode classification:** **Coupled observer**: a subject that otherwise stores and invokes the interface (`observer/OrderService.java:11–12,57–60`) instantiates concrete observers (`observer/OrderService.java:17–18`). No missing `OrderObserver` interface (`observer/OrderObserver.java:3–5`) or identity swap is evidenced.

**Verdict: MISAPPLIED — coupled observer.** (`observer/OrderService.java:11–23,57–60`.)

## 3. Template Method — processing sequence

| Check | Result | Source evidence |
|---|---|---|
| Abstract base processor? | **Pass** | `template/OrderProcessor.java:7`. |
| `processOrder()` declared `final`? | **Pass** | `template/OrderProcessor.java:8`. |
| Exactly five steps in the stated order? | **Pass** | `template/OrderProcessor.java:10–14` calls validation, total calculation, discount, payment, notification in sequence. |
| Custom steps implemented as protected/abstract hooks? | **Pass** | `template/OrderProcessor.java:17,24,28,30,35` defines protected methods including abstract discount and notification hooks. |
| Subclasses override hooks, not the entire pipeline? | **Pass** | `template/StandardOrderProcessor.java:5–14` and `template/PremiumOrderProcessor.java:5–15` override only `applyDiscount` and `sendNotification`; the pipeline remains in `template/OrderProcessor.java:8–15`. |

**Governing question — what varies, and who varies it?** The processing sequence does not vary (`template/OrderProcessor.java:8–15`). The discount and notification behavior do vary between standard (5% and ordinary confirmation, `template/StandardOrderProcessor.java:7–13`) and premium (15% and premium confirmation, `template/PremiumOrderProcessor.java:7–14`). The subclasses own those variations, so the inheritance skeleton is justified by actual differences instead of being empty ceremony.

**Mode classification:** No detected failure under missing interface, coupled observer, identity swap, or ceremony without need: the final skeleton is inherited (`template/OrderProcessor.java:7–15`; `template/StandardOrderProcessor.java:5–14`; `template/PremiumOrderProcessor.java:5–15`), and the differing hook behavior is visible in those subclasses.

**Verdict: CORRECT.** (`template/OrderProcessor.java:7–35`; `template/StandardOrderProcessor.java:5–14`; `template/PremiumOrderProcessor.java:5–15`.)

## Additional source-integrity findings (separate from pattern verdicts)

- `strategy/OrderNotifier.java:5` calls `Order.getCustomerEmail()`, but the available `strategy/Order.java:21–26` exposes `getId`, `getSubtotal`, `getTotal`, and `setTotal` only. That stale class is incompatible with this Order model.
- `observer/FixedAmountDiscount.java:16–17` and `observer/PercentageDiscount.java:17–18` call `Order.getSubtotal()`, while the current `observer/Order.java:22–43` instead defines `getOriginalTotal` and `getFinalTotal` (among other methods).
- `observer/PaymentProcessor.java:5` calls `Order.getTotal()`, which is absent from `observer/Order.java:22–43`.
- These are code consistency/build problems, not evidence that the Template Method pattern or the Observer subscription mechanism is structurally wrong. They should be addressed **only after** the raw output and this audit have been committed.

## Commit boundary

Do not edit generated Java sources before committing this audit. From the **repository root**:

```bash
git add lab7/pattern_audit.md
git commit -m "lab7: pattern audit"
```

The commit has **not** been run here; record its hash after committing locally.
