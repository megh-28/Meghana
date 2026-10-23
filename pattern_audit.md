# Lab 7 — Activity 2: Pattern Audit

**Evidence source:** `lab7(2).zip`, under `lab7/src/main/java/ie/ucd/comp41670/lab7/`. All citations below use paths relative to that Java package root and refer to **the unmodified uploaded source**. This is a static source review, not a passing-test claim. No production source was changed.

## 1. Strategy

| Check | Result | Source evidence |
|---|---|---|
| Shared `DiscountStrategy` interface with `apply(Order)` | **Pass** | `strategy/DiscountStrategy.java:5–7` defines the interface and `BigDecimal apply(Order order)`. |
| `OrderService` holds a reference to the strategy interface type only | **Fail (literal field/ownership requirement)** | `strategy/OrderService.java:6–12` stores only `PaymentProcessor` and `OrderNotifier`; `:14–16` receives a `DiscountStrategy` as a parameter rather than holding it in a field. This is **not in itself invalid Strategy**: method-parameter injection is still polymorphic dependency on the interface. |
| Fourth discount without editing `OrderService` | **Pass** | `strategy/OrderService.java:14,22` accepts and calls any `DiscountStrategy`; `strategy/Main.java:12–13` supplies two different implementations at call sites. A fourth implementer can be passed the same way. |
| `if`/`switch` on discount implementation type | **Pass** | `strategy/OrderService.java:14–29` delegates to `discountStrategy.apply(order)` at line 22; the `if` at line 23 checks the **payment result**, not the discount type. `strategy/Main.java:8–16` supplies concrete implementations without a discount-type branch. |

**Misapplication-mode classification.** The missing *stored field* is a deviation from the lab's stated check, but is **not** one of the four canonical misapplication modes: the shared interface exists (`strategy/DiscountStrategy.java:5–7`), and delegation still uses the interface (`strategy/OrderService.java:14,22`). In particular, it would be misleading to call this *missing interface* or *identity swap*. A stricter context-owns-strategy requirement would call for refactoring the constructor/field, not replacing the Strategy pattern.

**Governing question — what varies, and who varies it?** The algorithm varies between percentage and fixed discounts (`strategy/Main.java:12–13`); the caller chooses the implementation when invoking `OrderService.processOrder` (`strategy/OrderService.java:14`). This is not mere ceremony because two different algorithms are actually supplied via the same interface (`strategy/Main.java:12–13`; `strategy/DiscountStrategy.java:5–7`).

**Verdict: Misapplied against the assignment's explicit *held-reference* check (interface reference not stored); no canonical misapplication mode established.** The implementation still exhibits a valid Strategy delegation seam (`strategy/OrderService.java:14,22`).

## 2. Observer

| Check | Result | Source evidence |
|---|---|---|
| `OrderObserver` interface exists | **Pass** | `observer/OrderObserver.java:3–5` declares `update(String event, Order order)`. |
| Subject stores `List<OrderObserver>` | **Pass** | `observer/OrderService.java:11–12` declares and initializes it. |
| `subscribe()` accepts interface | **Pass** | `observer/OrderService.java:21–23` accepts an `OrderObserver` and adds it to the list. |
| Subject publishes through stored observers | **Pass** | `observer/OrderService.java:57–60` loops through `OrderObserver` references and calls `update` after setting `PLACED` at line 55. |
| Subject does not construct or directly reference concrete observers | **Fail** | `observer/OrderService.java:14–19` directly constructs `new EmailNotifier()` and `new LogNotifier()`. Those classes implement `OrderObserver` (`observer/EmailNotifier.java:4–10`; `observer/LogNotifier.java:3–9`). Same-package references require no explicit import, but still create concrete dependencies. |

**Misapplication-mode classification: coupled observer.** `OrderService` hardcodes both notifier implementations in its constructor (`observer/OrderService.java:14–19`), so default notification setup is coupled to concrete classes even though its storage and dispatch use the interface (`observer/OrderService.java:11–12,57–60`). This is precisely the *coupled-observer* mode. There is no missing-interface failure (`observer/OrderObserver.java:3–5`).

**Governing question — what varies, and who varies it?** Subscriber types vary: the package contains email and log observers (`observer/EmailNotifier.java:4–10`; `observer/LogNotifier.java:3–9`). External callers can subscribe additional observers through `subscribe(OrderObserver)` (`observer/OrderService.java:21–23`), but the subject currently chooses and constructs two defaults itself (`observer/OrderService.java:14–19`). The interface and subscription mechanism serve a real variation point, rather than ceremony (`observer/OrderObserver.java:3–5`; `observer/OrderService.java:21–23`).

**Verdict: Misapplied — coupled observer** (`observer/OrderService.java:14–19`). Refactoring should move concrete observer creation outside the subject while retaining subscriptions and event delivery (`observer/OrderService.java:21–23,57–60`).

## 3. Template Method

| Check | Result | Source evidence |
|---|---|---|
| Abstract `OrderProcessor` with final `processOrder()` | **Pass** | `template/OrderProcessor.java:7–8` declares abstract class and `public final void processOrder(Order order)`. |
| Exact five-step sequence | **Pass** | `template/OrderProcessor.java:10–14` calls validate, calculate total, apply discount, process payment, send notification in order. |
| Customizable steps are abstract/protected hooks | **Pass** | `template/OrderProcessor.java:17,24,28,30,35` defines protected hooks and protected abstract variation points. |
| Subclasses customize hooks, not full pipeline | **Pass** | `template/StandardOrderProcessor.java:5–14` and `template/PremiumOrderProcessor.java:5–15` override `applyDiscount` and `sendNotification`; the final pipeline remains in `template/OrderProcessor.java:8–15`. |

**Misapplication-mode classification:** No failed structural check, so none of the four modes is demonstrated: the actual implementation uses inheritance and a final skeleton (`template/OrderProcessor.java:7–15`), not a composition-based *identity swap*.

**Governing question — what varies, and who varies it?** Discounts and confirmation wording vary between Standard and Premium processors (`template/StandardOrderProcessor.java:7–13`; `template/PremiumOrderProcessor.java:7–14`). The subclasses own that variation through hook overrides; the abstract base fixes the five-step sequence (`template/OrderProcessor.java:8–15,28,35`). With two concrete variants, the pattern has a demonstrated variation point rather than obvious ceremony (`template/StandardOrderProcessor.java:5–14`; `template/PremiumOrderProcessor.java:5–15`).

**Verdict: Correct** (`template/OrderProcessor.java:7–35`; `template/StandardOrderProcessor.java:5–14`; `template/PremiumOrderProcessor.java:5–15`).

## 4. Additional source-integrity finding (outside the pattern-mode taxonomy)

The uploaded `observer` package combines the new, simplified `Order` model with older classes that expect a different API. `observer/Order.java:7–44` offers `getOriginalTotal()` and `getFinalTotal()` but no `getSubtotal()` or `getTotal()`; meanwhile `observer/FixedAmountDiscount.java:16–18` and `observer/PercentageDiscount.java:16–19` call `getSubtotal()`, and `observer/PaymentProcessor.java:4–6` calls `getTotal()`. Thus the **complete Observer package as uploaded cannot compile unchanged**. This is a source/API mismatch, *not* any of the four design-pattern misapplication modes. It must be recorded separately and fixed during the later refactor, without rewriting the raw-generation commit. These are static findings; no Maven test run is claimed.

## 5. Misapplication-mode summary

| Mode | Source-based result |
|---|---|
| Ceremony without need | Not established: both discount alternatives are used (`strategy/Main.java:12–13`); email and log handlers exist (`observer/EmailNotifier.java:4`; `observer/LogNotifier.java:3`); Standard/Premium hooks differ (`template/StandardOrderProcessor.java:7–13`; `template/PremiumOrderProcessor.java:7–14`). |
| Missing interface | Not found: `strategy/DiscountStrategy.java:5–7` and `observer/OrderObserver.java:3–5` exist. |
| Coupled observer | **Found:** `observer/OrderService.java:14–19` constructs concrete notifiers. |
| Identity swap | Not found in Template Method: `template/OrderProcessor.java:7–15` uses an inherited final skeleton, and subclasses override hooks (`template/StandardOrderProcessor.java:5–14`; `template/PremiumOrderProcessor.java:5–15`). |

## 6. Audit-before-refactor checkpoint

Save this file as `lab7/pattern_audit.md` and commit it **before editing any of the Java files**. Run these commands from the portfolio repository root:

```bash
git add lab7/pattern_audit.md
git commit -m "lab7: pattern audit"
```

Do not describe this commit as completed until Git reports success. The uploaded Java files remain unchanged by this audit.
