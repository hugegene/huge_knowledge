# Spec: Cart percentage discount

> **Owner: DEV.** QA may read this, never edit it. This is the intent — the root
> source of truth for the feature. (Sample feature demonstrating the DEV–QA
> contract; see `dev-qa-contract-agentic-era.md`.)

## Intent

Provide `apply_discount(subtotal, percent) -> amount_payable`: the amount due
after applying a percentage discount to a cart subtotal.

## Behaviour

1. `subtotal` is a non-negative currency amount. `percent` is a number in the
   closed range `[0, 100]`.
2. `amount_payable = subtotal − (subtotal × percent ÷ 100)`.
3. The result is rounded to **2 decimal places using banker's rounding**
   (round-half-to-even).
4. `percent = 0` returns the subtotal unchanged; `percent = 100` returns `0.00`.

## Errors

5. `subtotal < 0` → raise `ValueError`.
6. `percent < 0` or `percent > 100` → raise `ValueError`.
7. Non-numeric `subtotal` or `percent` → raise `TypeError`.

## Out of scope

Tax, currency conversion, and stacking multiple discounts. A request for any of
these is a **spec change** (hand back to DEV), not a conformance fix.
