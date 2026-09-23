# Part 4A

## Part A
### AI's Reply

The test `quotesDiscountForLoyalCustomer()` expected a 5-year customer’s $100 order to receive a 10% discount, producing $90.00. The code instead returned $95.00, which corresponds to only a 5% discount.

The stack trace shows:

- The failing test is `PriceEngineTest.quotesDiscountForLoyalCustomer()`.
- The failure occurs at `PriceEngineTest.java:92`.
- JUnit reports the mismatch: expected `90.0`, received `95.0`.
- The remaining stack frames are JUnit reflection and assertion internals.

The most likely root cause is an off-by-one error in the loyalty-tier boundary logic: a customer with exactly 5 years of loyalty is being treated as tier 2 instead of tier 3. Likely, the code checks for more than 5 years rather than including 5 years.

    JUnit Jupiter:PriceEngineTest:quotesDiscountForLoyalCustomer()

        MethodSource [className = 'PriceEngineTest', methodName = 'quotesDiscountForLoyalCustomer', methodParameterTypes = '']

        => org.opentest4j.AssertionFailedError: 5-year loyalty customer should get 10% off (tier 3) ==> expected: <90.0> but was: <95.0>

This was enough for the AI to identify the root cause of the error.

The explanation given by AI was correct.

## Part C

### Most Useful

The patch reply was most useful because it identified the exact boundary error and proposed the change.

### Least Useful

The explanation reply was less useful because it mainly restated information already visible in the test failure.

The full stack trace gives the AI useful context: the failing test name, source line, expected value, actual value, and assertion type. However, most of the lower stack frames are JUnit implementation details and add noise. Pasting the full trace can consume context and make the important failure harder to find.

The best amount is usually the failure summary, the assertion message, the expected and actual values, and the relevant source/test lines. One or two lines alone may be insufficient, especially if they omit the test name or source location.

# Part 4B

## Setup

Cyclomatic complexity is 23.

Two SpotBugs findings are:

- `M V EI2: new Order(long, List, boolean, String)` may expose internal representation by storing an externally mutable object into `lines` at `Order.java:[line 4]`
- `M V EI: Order.lines()` may expose internal representation by returning `lines` at `Order.java:[line 4]`

## Part A

| Priority | Method | Cyclomatic complexity | First refactor move | Expected reduction |
|---|---|---:|---|---|
| 1 | `PriceEngine.quote(Order, Customer)` | 23 | `extract-method` | Extracting the loyalty-tier, promo, or tax logic would move several branches out of `quote`, reducing its complexity substantially, likely below PMD’s threshold. |

There are no other explicitly defined methods in this file. The constructor is implicit and does not require refactoring.

## Part B

After refactoring loyalty discount, the cyclomatic complexity is 19.

### Refactor

    private static Money applyLoyaltyDiscount(Money running, Money subtotal, Customer customer) {
        int years = customer.loyaltyYears();
        double loyaltyRate;
        if (years >= 5) {
            loyaltyRate = 0.10;
        } else if (years >= 3) {
            loyaltyRate = 0.05;
        } else if (years >= 1) {
            loyaltyRate = 0.02;
        } else {
            loyaltyRate = 0.0;
        }
        if (loyaltyRate > 0.0) {
            return running.subtract(subtotal.times(loyaltyRate));
        }
        return running;
    }

## Part C

Change made in `Order.java`:

    public Order {
        lines = (lines == null) ? null : List.copyOf(lines);
    }

All the tests passed and both the bugs vanished from the output of SpotBugs.

## Part D

- Cyclomatic complexity dropped from 23 to 19, a reduction of 4.
- The SpotBugs fix did not require a deep understanding of the rule beyond the AI explanation. The explanation was sufficient to proceed because it clearly identified the mutable-list exposure and the standard fix.
- One refactor I rejected was extracting all of the promo, loyalty, and tax calculations out of `PriceEngine.quote` into multiple helper methods. I rejected it because it broadened the scope beyond the actual issue and was not necessary to fix the bug or satisfy the finding.