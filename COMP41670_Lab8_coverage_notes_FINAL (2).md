# Lab 8 - Coverage Notes

## Activity 1: Raw AI-Generated Test Suite

### Overall Coverage

- Line Coverage: 98.84% (85/86 lines)
- Branch Coverage: 97.5% (39/40 branches)

### Class-wise Coverage

| Class | Instruction Coverage | Branch Coverage |
|---|---|---|
| Product | 100% | 100% |
| OrderLine | 100% | N/A |
| OrderStatus | 100% | N/A |
| Order | 100% | 100% |
| OrderBook | 100% | 100% |
| PricingCalculator | 90% | 87% |
| **Overall** | **98%** | **97%** |

### JaCoCo Report

![Initial JaCoCo Coverage](evidence/final.png)

## Activity 2 — Reviewing the AI-generated tests

I checked each generated test to see whether it could actually fail, whether it tried any boundary values, whether it handled unusual inputs (empty, negative, null or duplicate values), and whether the assertions checked something useful. I counted a test as *load-bearing* if it could catch at least one real mistake. A test can still be load-bearing even if it misses other important cases.

### ProductTest.java — 4 tests

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `createsProductWithValidValues` | Yes | No | No | Checks the exact SKU, name and price; does not try a very large price. | Load-bearing |
| `rejectsNullOrBlankSku` | Yes | No numeric boundary | Null, empty and whitespace-only SKU | Uses `assertThrows` to check that invalid SKUs are rejected. | Load-bearing |
| `rejectsNullOrBlankName` | Yes | No numeric boundary | Null, empty and whitespace-only name | Checks three invalid-name cases using expected exceptions. | Load-bearing |
| `rejectsNonPositivePrice` | Yes | Zero and -1 | Zero and negative price | Checks the price rule at and immediately below zero. | Load-bearing |

### OrderLineTest.java — 2 tests

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `exposesProductSkuAndUnitPrice` | Yes | No; quantity is 3 | No | Checks the product, quantity, SKU and unit price against exact values. | Load-bearing |
| `recordEqualityUsesProductAndQuantity` | Yes | No; quantity is 2 | Two equivalent records | Checks equal records, but never checks that different products or quantities are unequal. | Load-bearing, limited |

### OrderStatusTest.java — 1 test

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `containsAllLifecycleStatusesInOrder` | Yes | No | No | Checks the exact enum values and their order, but not whether transitions between them are valid. | Load-bearing, limited |

### OrderTest.java — 9 tests

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `startsPendingAndEmptyWithItsIdentifiers` | Yes | Initial state | Empty order | Checks both IDs, `PENDING` status and empty lines. | Load-bearing |
| `rejectsNullOrBlankCustomerId` | Yes | No numeric boundary | Null, empty and whitespace-only ID | Expects an exception for each invalid ID. | Load-bearing |
| `addsLinesAndPreservesFirstAddedSkuOrder` | Yes | No; uses quantities 2 and 3 | Two distinct SKUs | Compares the complete ordered list, but does not try a repeated SKU. | Load-bearing |
| `rejectsInvalidLinesWithoutChangingOrder` | Yes | Quantity 0 and 100 | Null product; invalid quantities | Exceptions are meaningful. However, `originalLines` is a view of the same underlying list, so comparing it with `getLines()` is not a reliable check that the order stayed unchanged. | Load-bearing, limited |
| `combinesLinesWithTheSameSkuAndRejectsCombinedQuantityOverLimit` | Yes | Combined quantity 99, then 100 | Repeated SKU with different product details | Checks the merged line and the limit. **Failed in the original run.** | Load-bearing |
| `returnedLinesCannotBeModified` | Yes | No | Attempt to clear returned list | `assertThrows` is meaningful because the returned list is supposed to be unmodifiable. | Load-bearing |
| `removesLineAndRejectsUnknownSku` | Yes | One line left | Unknown SKU | Checks the exact remaining line and the exception for an unknown SKU. | Load-bearing |
| `paysOnlyNonEmptyPendingOrdersAndShipsOnlyPaidOrders` | Yes | Before/after state transitions | Empty order and invalid transitions | Checks status changes and exceptions, not just whether the methods run. | Load-bearing |
| `cancelsPendingOrPaidOrdersButNotShippedOrAlreadyCancelledOrders` | Yes | Different lifecycle states | Repeated cancellation; shipped order | Checks permitted and forbidden cancellation. **Failed in the original run** when a shipped order was cancelled. | Load-bearing |

### OrderBookTest.java — 5 tests

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `createsAndStoresOrdersWithSequentialIds` | Yes | First and second generated IDs | Two different customers | Checks exact IDs, lookup results and distinct order objects. | Load-bearing |
| `findReturnsEmptyWhenIdIsUnknown` | Yes | No | Unknown ID | Explicitly checks `Optional.empty()`. | Load-bearing |
| `rejectsInvalidCustomerIdAndDoesNotStoreAnOrder` | Yes | First valid ID after failed creations | Null and blank customer ID | Checks exceptions and confirms invalid attempts did not consume the first order ID. | Load-bearing |
| `returnsCustomerOrdersInCreationOrder` | Yes | No | Repeated customer ID; unknown customer | Checks exact order and an empty list for an unknown customer. | Load-bearing |
| `returnedCustomerOrderListCannotBeModified` | Yes | No | Attempt to clear returned list | The exception assertion checks the read-only result. | Load-bearing |

### PricingCalculatorTest.java — 6 tests

| Test | Could fail? | Boundary checks | Empty / negative / duplicate inputs | Assertion quality | Verdict |
|---|---|---|---|---|---|
| `calculatesLineAmount` | Yes | No; quantity is 3 | No | Checks the precise amount of 3750 cents. | Load-bearing |
| `appliesTenPercentDiscountAtTenOrMoreUnitsAndRoundsDown` | Yes | 9 and 10 units | One-cent unit price | Tests the discount threshold and rounding using exact expected amounts. **Failed originally.** | Load-bearing |
| `calculatesDiscountedSubtotalAndLeavesOrderUnchanged` | Yes | Exactly 10 units | Two different lines | Checks an exact subtotal, but the AI's expected **1109 cents is incorrect**: §3.2–§3.3 imply **1309 cents**. Its 'unchanged' checks inspect only line count and non-emptiness, not full contents. **Failed originally.** | Load-bearing, limited |
| `calculatesShippingAtFreeShippingThresholdAndForEmptyOrders` | Yes | 4999 and 5000 cents | Empty order | Checks both sides of the free-shipping threshold and the empty case. | Load-bearing |
| `roundsVatHalfUpAndDoesNotIncludeShipping` | Yes | Half-cent rounding case | No | Checks exact VAT amounts rather than only checking for a positive result. **Failed originally.** | Load-bearing |
| `calculatesTotalFromSubtotalShippingAndVat` | Yes | No | No | Checks an exact final total of 618 cents, but not an order with a positive bulk discount. | Load-bearing |

### Test review tally

| Result | Count |
|---|---:|
| Tests reviewed | **27** |
| Load-bearing tests | **27** |
| Completely tautological / assertion-free tests | **0** |
| Tests passed in the original run | **22** |
| Tests failed in the original run | **5** |

I counted all 27 as load-bearing because each has at least one assertion that could detect incorrect behaviour. That surprised me a little: the generated suite is not just a collection of `assertNotNull` checks. Still, some tests are incomplete. For example, the record-equality test checks only the equal case, and the 'unchanged order' checks are weaker than they first appear. The five failing tests are worth investigating rather than dismissing automatically. Some reveal genuine disagreements with `SPEC.md`, while the discounted-subtotal test also contains an incorrect AI-generated expected value: **1109** rather than **1309** cents (`101 × 10 − 101 + 200 × 2`). This is a weakness in the test itself, separate from the implementation's discount-threshold bug.

## Activity 2 — PIT mutation testing

### How I ran PIT

My first PIT attempt failed because five generated tests already failed on the unmodified starter. PIT needs a passing baseline before it can evaluate mutations.

For the mutation-testing experiment, I temporarily added `@Disabled` to those five tests. I did not change their assertions or the production code. The five tests were skipped **only in the PIT experiment**, leaving **22 passing tests** for PIT to analyse. The original 27-test suite is the reference version that must be retained in the repository.

The first successful PIT run used the default mutators and killed 43 out of 50 mutations, leaving three survivors. I then ran a broader experiment with the following temporary setting in `build.gradle`:

```groovy
mutators = ['STRONGER', 'INLINE_CONSTS']
```

The results and five survivors below all come from **that single expanded run**. They should not be read as results for the full set of 27 tests; the five originally failing tests were excluded to make PIT runnable. This was a temporary testing configuration, not a change to the expected behaviour.

### PIT results — expanded run

| Measure | Result |
|---|---:|
| Tests enabled for PIT | **22** (5 temporarily disabled) |
| Total mutations | **80** |
| Killed | **68** |
| Survived | **5** |
| No coverage | **7** |
| **Mutation score** | **85% (68/80)** |
| Test strength | **93% (68/73 covered mutations)** |
| PIT line coverage (mutated classes) | **93% (68/73)** |

### Breakdown by class

| Class | PIT line coverage | Killed / total | Survived | No coverage |
|---|---:|---:|---:|---:|
| `Order.java` | 36/40 (90%) | 31/34 | 1 | 2 |
| `OrderBook.java` | 15/15 (100%) | 10/10 | 0 | 0 |
| `OrderLine.java` | 3/3 (100%) | 2/2 | 0 | 0 |
| `PricingCalculator.java` | 14/15 (93%) | 25/34 | 4 | 5 |
| **Total** | **68/73 (93%)** | **68/80 (85%)** | **5** | **7** |

### Five surviving mutants

#### 1. `Order.java`, line 53 — changed conditional boundary

**Original code:**

```java
if (quantity < MIN_QUANTITY || quantity > MAX_QUANTITY) {
```

**What PIT changed:** One of the quantity comparisons was changed at its boundary. The intended range in `SPEC.md` §2.2 is **1–99 inclusive**.

**Why it survived:** The enabled suite checked invalid quantities 0 and 100 but did not explicitly test that a single line with quantity 99 is accepted. The only generated test reaching a combined quantity of 99 was disabled for PIT. PIT reports which source line and mutation type survived, but the screenshot does not show which of the two comparisons was changed. Testing both valid endpoints would remove that uncertainty.

**Test that should catch it:** A focused boundary test that accepts quantities **1 and 99** and rejects **0 and 100**, comparing the resulting order contents as well.

#### 2. `PricingCalculator.java`, line 19 — condition replaced with `false`

**Original code:**

```java
if (line.quantity() > BULK_THRESHOLD) {
```

**What PIT changed:** PIT replaced the condition with `false`, so this method would never apply a bulk discount.

**Why it survived:** None of the enabled tests checked a case where the discount should be greater than zero. The AI-generated discount-specific test was one of the five temporarily disabled tests.

**Test that should catch it:** Check `lineDiscount()` for **11 units at 100 cents each**. Under §3.2, the expected discount is **110 cents**, whereas this mutant would give 0.

#### 3. `PricingCalculator.java`, line 19 — constant `10` replaced with `11`

**What PIT changed:** The quantity threshold used in the conditional changed from 10 to 11. Since the original code compares using `>`, the mutant would not give a discount until the quantity reached 12.

**Why it survived:** The enabled tests did not check the distinction between 11 and 12 units.

**Test that should catch it:** Check **11 units at 100 cents each**, expecting **110 cents** off under `SPEC.md` §3.2. The original implementation would pass that particular case; the threshold-11 mutant would not.

#### 4. `PricingCalculator.java`, line 19 — changed conditional boundary

**What PIT changed:** The comparison changed from `> 10` to `>= 10`, so a discount would also apply at exactly 10 units.

**Why it survived:** The test that checks exactly 10 units was temporarily disabled, leaving no enabled test that distinguished these conditions.

**Test that should catch it:** This one needs care. A test expecting **no discount** at 10 units would distinguish and kill the mutant against the starter, but it would contradict `SPEC.md` §3.2. The specification says **10 or more**, so the mutant actually fixes the starter's threshold behaviour. The existing AI-generated test for exactly 10 units is the right specification-based test: it fails against the original starter and would pass this mutant. I would not write an incorrect expectation just to kill it.

#### 5. `PricingCalculator.java`, line 28 — subtraction replaced with addition

**Original code:**

```java
subtotal += lineAmount(line) - lineDiscount(line);
```

**What PIT changed:** The subtraction became addition. Instead of subtracting the discount, the subtotal would increase by the discount amount.

**Why it survived:** The enabled tests effectively exercised cases with a zero discount, so adding or subtracting the discount produced the same result. The more relevant discounted-subtotal test was temporarily disabled.

**Test that should catch it:** Check a line of **11 units at 1000 cents each**. The amount is 11,000 cents and the 10% discount is 1,100 cents, so the expected subtotal is **9,900 cents**. The mutant would give **12,100 cents**. This would expose the incorrect arithmetic without changing the production code.

### What the mutation results say about coverage

The original JaCoCo report looked very strong: **85 out of 86 lines (98.84%)** and **39 out of 40 branches (97.5%)** were covered. But those figures only show which parts of the code ran. They do not tell me whether the tests would notice an incorrect result.

For PIT, I had to use a smaller, passing subset of 22 tests. That run covered **68 out of 73 lines (93%)** in the mutated classes, so the PIT line-coverage figure should not be treated as a direct like-for-like comparison with the original JaCoCo report. Even within that covered code, five mutations survived.

The examples make the gap clearer. In `Order.java:53`, the quantity check ran, but the enabled tests did not assert acceptance of every valid boundary, especially 99. In `PricingCalculator.java:19`, changing the bulk-discount condition went unnoticed because no enabled test checked a positive discount at the important threshold values. In `PricingCalculator.java:28`, addition and subtraction gave the same result when the discount was zero, so the wrong operator escaped detection.

What I learned is that **running a line is not the same as testing its behaviour**. I also need to interpret mutations against the specification: the `> 10` to `>= 10` mutant survived even though it moves the code closer to the specified behaviour.

### PIT report evidence

The following screenshots are from the **expanded** run described above:

- [PIT project summary](evidence/pit_summary.png)
- [PIT mutations in Order.java](evidence/pit_order.png)
- [PIT mutations in PricingCalculator.java](evidence/pit_pricing.png)

The original failing tests and the temporary PIT exclusions are separate from the behaviour that the specification requires. No production code fixes are part of Activity 2.
