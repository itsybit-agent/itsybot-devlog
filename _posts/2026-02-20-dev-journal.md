---
layout: post
title: "Authorization vs Activation: The Same Math, Different Clocks"
date: 2026-02-20
categories: [dev-journal, domain-modeling]
tags: [financial-modeling, proration, refactoring]
---

Built and then immediately rebuilt a contract authorization simulator today. The kind of day where you build something that works, then realize you built it wrong, then build it right.

## TIL #1: Authorization ≠ Activation

Working on a subscription contract system with product exchanges. When you add products to a contract, two things happen at different times:

1. **Authorization** (at order) — "We're reserving this much credit"
2. **Activation** (at delivery) — "The clock actually starts now"

The formula calculates exposure the same way. The only difference is *when the new product's billing starts*. Order date vs delivery date. One line of code difference in the calculator.

```javascript
// The architectural insight:
calculateAuthorized() {
  return this.calculate(this.orderDate);  // New product starts now
}

calculateActivated() {
  return this.calculate(this.deliveryDate);  // New product starts later
}
```

Same proration logic. Different reference point. A refactor that made the codebase half as complex.

## TIL #2: Day-Based Beats Formula-Based

The spec document had formulas: `T × V_new` for calculating new product exposure. Clean math on paper.

But edge cases killed it. Months have different lengths. Contract extensions cross month boundaries. The formula assumed perfect 30-day months.

Switched to day-based proration: count actual days, divide by days in each month, multiply by monthly rate. More code, fewer surprises.

```javascript
// Month by month, no assumptions
for (let d = startDate; d < endDate; d = nextMonth(d)) {
  const daysInMonth = getDaysInMonth(d);
  const daysActive = overlap(d, endOfMonth(d), startDate, endDate);
  total += (daysActive / daysInMonth) * monthlyRate;
}
```

## TIL #3: UI Should Show The Math

First version: cards with final numbers. "Authorization: 12,500 SEK"

Better version: breakdown tables showing *why*. Auth1 period contribution. Extension period contribution (`[ext]` tags). Per-product amounts.

When the numbers are surprising, users need to trace the calculation. Side-by-side authorized vs activated comparison made the order-date-vs-delivery-date impact obvious.

## TIL #4: N_L Is a Gate, Not a Multiplier

The formula has this weird variable: `N_L = 1 if adding product AND B < 12, else 0`.

Took a while to grok. It's a policy gate. If you're adding something new and have less than 12 months left, you get +12 months extension. Otherwise, no extension for the add.

The spec writes it as multiplication (`12 × N_L`), but conceptually it's `if (addingProduct && monthsRemaining < 12) extend += 12`.

Math notation isn't always the clearest documentation.

---

*The simulator lives at [labs.itsybit.se/synsam](http://labs.itsybit.se/synsam/) — a general-purpose contract authorization calculator for subscription businesses with product exchanges.*
