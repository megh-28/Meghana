# Lab 8 — Spec-First Test List 

## §1 — Money and products

| Test name | Expected behaviour | Clause |
|---|---|---|
| `storesProductPriceInWholeCents` | A product stores its unit price as the supplied `long` number of euro cents. | §1.1 |
| `rejectsNullOrBlankSkuAndName` | Creating a product with a null or blank SKU or name throws `IllegalArgumentException`. | §1.2 |
| `rejectsZeroOrNegativeUnitPrice` | Creating a product with a price of zero or less throws `IllegalArgumentException`. | §1.2 |
| `treatsProductsWithTheSameSkuAsOneOrderProduct` | Adding products with the same SKU does not create separate order lines, even if other product details differ. | §1.3, §2.3 |

## §2 — Orders and order lines

| Test name | Expected behaviour | Clause |
|---|---|---|
| `newOrderStartsPendingAndEmpty` | A new order has status `PENDING` and contains no lines. | §2.1 |
| `rejectsNullOrBlankOrderCustomerId` | Creating an order with a null or blank customer ID throws `IllegalArgumentException`. | §2.1 |
| `acceptsMinimumAndMaximumLineQuantity` | Adding a line with quantity 1 or 99 succeeds while the order is `PENDING`. | §2.2 |
| `rejectsOutOfRangeQuantitiesWithoutChangingOrder` | Quantities 0 and 100 cause `IllegalArgumentException` and leave existing lines unchanged. | §2.2 |
| `rejectsNullProductWithoutChangingOrder` | Adding a null product throws `IllegalArgumentException` and leaves the order unchanged. | §2.2 |
| `mergesRepeatedSkuIntoSingleLine` | Adding the same SKU twice increases the quantity of the existing line instead of creating another line. | §2.3 |
| `acceptsCombinedQuantityOfExactlyNinetyNine` | Repeated additions bringing one SKU's quantity to exactly 99 are accepted. | §2.3 |
| `rejectsCombinedQuantityAboveNinetyNineWithoutChanges` | A repeated-SKU addition that would make the quantity exceed 99 throws `IllegalArgumentException` and preserves existing lines. | §2.3 |
| `removesAnExistingSkuLine` | Removing an existing SKU removes its line from the order. | §2.4 |
| `rejectsRemovingAnUnknownSku` | Removing a SKU not present in the order throws `IllegalArgumentException`. | §2.4 |
| `rejectsLineChangesOutsidePending` | Adding or removing lines in `PAID`, `SHIPPED` or `CANCELLED` throws `IllegalStateException` and leaves the order unchanged. | §2.5 |
| `preservesFirstSkuInsertionOrder` | `getLines()` lists SKUs in their first-added order, even after another quantity is added to an existing SKU. | §2.6 |
| `returnsUnmodifiableOrderLines` | A caller cannot change the list returned by `getLines()`. | §2.6 |

## §3 — Pricing

| Test name | Expected behaviour | Clause |
|---|---|---|
| `calculatesLineAmountInCents` | Line amount equals unit price in cents multiplied by quantity. | §3.1 |
| `doesNotDiscountNineUnits` | A line with 9 units receives no bulk discount. | §3.2 |
| `appliesDiscountStartingAtTenUnits` | A line with exactly 10 units receives a 10% discount. | §3.2 |
| `roundsBulkDiscountDownToWholeCent` | Any fractional-cent bulk discount is rounded down. | §3.2 |
| `addsDiscountedLineAmountsForSubtotal` | Subtotal is the sum of each line amount after its bulk discount. | §3.3 |
| `returnsZeroSubtotalForEmptyOrder` | An order with no lines has subtotal zero. | §3.3 |
| `returnsZeroShippingForEmptyOrder` | An order with no lines has no shipping charge. | §3.4 |
| `chargesShippingBelowFiveThousandCents` | A non-empty order with subtotal below 5000 cents is charged 495 cents for shipping. | §3.4 |
| `givesFreeShippingAtFiveThousandCents` | A non-empty order with subtotal of exactly 5000 cents or more receives free shipping. | §3.4 |
| `roundsVatHalfUp` | VAT is 23% of subtotal, with a half cent rounded up to the nearest whole cent. | §3.5 |
| `excludesShippingFromVat` | Shipping does not increase the VAT amount. | §3.5 |
| `calculatesTotalAsSubtotalShippingPlusVat` | Order total equals subtotal plus shipping plus VAT. | §3.6 |
| `pricingDoesNotChangeOrderLinesOrStatus` | Calling the pricing methods leaves the order's lines and status unchanged. | §3 (intro) |

## §4 — Order lifecycle

| Test name | Expected behaviour | Clause |
|---|---|---|
| `payChangesNonEmptyPendingOrderToPaid` | Paying a non-empty `PENDING` order changes its status to `PAID`. | §4.1 |
| `rejectsPayingEmptyOrderWithoutChangingStatus` | Paying an empty order throws `IllegalStateException` and leaves it `PENDING`. | §4.1 |
| `shipChangesPaidOrderToShipped` | Shipping a `PAID` order changes its status to `SHIPPED`. | §4.2 |
| `cancelChangesPendingOrPaidOrderToCancelled` | Cancelling a `PENDING` or `PAID` order changes its status to `CANCELLED`. | §4.3 |
| `rejectsCancellingShippedOrCancelledOrder` | Cancelling a `SHIPPED` or already `CANCELLED` order throws `IllegalStateException` without changing status. | §4.3, §4.4 |
| `rejectsOtherInvalidStatusTransitions` | Any pay, ship or cancel transition not listed by the specification throws `IllegalStateException` and preserves status. | §4.4 |

## §5 — The order book

| Test name | Expected behaviour | Clause |
|---|---|---|
| `createsAndStoresOrdersWithSequentialIds` | Each call to `create` stores an order, returning IDs beginning `ORD-0001`, `ORD-0002`, etc. | §5.1 |
| `findReturnsStoredOrderOrEmptyOptional` | `find(id)` returns the stored order when present and an empty `Optional` otherwise. | §5.2 |
| `listsCustomerOrdersInCreationOrder` | `ordersFor(customerId)` returns only that customer's orders, ordered by creation time, or an empty list. | §5.3 |
| `returnsUnmodifiableCustomerOrderList` | A caller cannot modify the list returned by `ordersFor(customerId)`. | §5.3 |

**Total reference scenarios:** 40.
