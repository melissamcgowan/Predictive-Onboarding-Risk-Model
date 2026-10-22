# Predictive Onboarding Risk Model

An interactive web app that predicts which new customers are likely to struggle during onboarding, using only the signals available in their first 30 days. It ranks the queue by risk, shows exactly which signals drive each score, and recommends the play a CSM should run first.

All accounts, usage numbers, and outcomes are **synthetic**. No real customer data is included.

**Open `index.html` in any browser. No build step and no dependencies.**

## The problem

Onboarding problems surface late. By the time an account misses its first renewal conversation or goes quiet, the cause (an unconnected integration, a sponsor who never showed up, seats nobody activated) was visible weeks earlier. This model catches those patterns at the day-30 checkpoint, while there is still time to fix them.

It pairs with the Onboarding Time-to-Value Tracker. The tracker measures milestone pace. This model predicts who is likely to miss those milestones.

## How the score works

Each signal is compared with a **segment-specific target** and converted into a shortfall between 0 (on target) and 1 (far off target). The risk score is the weighted average shortfall, scaled to 0-100.

| Signal | Default weight | Enterprise target | SMB target |
|---|---|---|---|
| Login activity (logins per user per week) | 20 | 4 | 3 |
| Core feature adoption | 15 | 60% | 50% |
| Seat activation | 15 | 60% | 80% |
| Integration connected | 15 | Required | Optional (half penalty) |
| Training attendance | 10 | 80% | 80% |
| Admin responsiveness (days to reply) | 10 | 2 days | 1.5 days |
| Support friction (tickets, high severity) | 10 | Scaled | Scaled |
| Executive sponsor engaged | 5 | Yes | Yes |

**Tiers** (adjustable in the app): High at 45 and above, Medium at 28 and above, Low below that.

## What you can do in the app

- **Risk queue.** Ranked list of accounts reaching day 30. Filter by segment, sort by risk, ARR, or name.
- **Account drill-down.** Click any account to see its score broken into points per signal, the raw value against target, and up to three recommended plays in priority order.
- **Adjustable model.** Change signal weights and tier cutoffs. The queue and the backtest both recalculate instantly.
- **Backtest.** 90 past synthetic accounts with known day-90 outcomes. Shows precision, recall, a confusion matrix, the struggle rate by tier, and the score distribution split by outcome.

## Recommended plays

Each signal maps to one concrete play, for example a connection session for an unconnected integration, a seat activation campaign with a named rollout lead, or an executive alignment call when the sponsor is missing. The app lists the plays for the account's three biggest risk drivers.

## Two-layer design

| Layer | What it does | Swap it for |
|---|---|---|
| Data layer (`makeData`) | Generates synthetic accounts and outcomes from a seeded random generator | A CSP export or product-usage feed |
| Model layer (`scoreAccount`, `backtest`) | Pure functions with no DOM access | Nothing. Reuse as is |

Replacing the data layer with real exports runs the same model on live accounts. The model functions are pure, so they are easy to test and to port to Python or SQL.

## Calibrating on real data

1. Export 6 to 12 months of past onboardings with their day-30 signals and a day-90 outcome (adoption target hit, or cancellation request).
2. Load them into the backtest and adjust the weights until the High tier captures the accounts that struggled.
3. Set the cutoffs so the flagged group is a size your team can actually work.

## Limitations

- Weights are expert-set defaults, not fitted. A production version would fit them with logistic regression on historical outcomes.
- Targets are static per segment. Real programs often vary them by product tier or use case.
- The backtest uses synthetic outcomes, so its accuracy numbers demonstrate the method, not real-world performance.
