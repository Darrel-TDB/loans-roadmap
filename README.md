# Various Calculators

A single-file, no-build calculator with four tools:

0. **SIL Calculator + Comparison** — computes Downpayment, Loan Amount, add-on rate, Regular EMI, Processing Fee, VAS Fee, and Total EMI for an "Applied" and a "Counter Offer" scenario side by side, with the rate card (segment/RBP/term → add-on rate, fee rate, DP band, LA band) built into the page. Flags DP rates outside a product's allowed band and terms that aren't offered for a product, and shows how much the Counter Offer's Total EMI differs from the Applied Total EMI.
1. **Full Payment Settlement** — computes the total payoff amount for a loan as of any date: outstanding balance + accrued interest since the last payment + any uncollected "Beginning Accrual" from origination.
2. **Amortization Schedule & EMI** — builds a full monthly amortization schedule from Loan Amount, Term, and APR, handling the stub period between the Date Booked and Date of 1st Collection the same way.
3. **Solar Financing** — builds the solar-loan amortization schedule and EMI, with APR set automatically by term (24/36/48 months), a fixed 3% processing fee and fixed 4% VAS/PayHinga fee (both spread across the term), a baseline-days cap on the 1st period's interest, and the excess deferred to the final payment.

Day-count basis for tabs 1–3: **Actual/365**.

## Files

- `index.html` — the calculator itself (logo is embedded inline, no external image file needed)
- `README.md` — this file

## How to use it

Just open `index.html` in a browser — no installation, no build step, no dependencies beyond a Google Fonts CDN call (it still works offline, it'll just fall back to system fonts).

### Hosting on GitHub Pages
1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Set source to the branch containing `index.html` (root folder).
4. Your calculator will be live at `https://<username>.github.io/<repo>/`.

## The logic, in brief

**SIL Calculator + Comparison** looks up a rate-card row by General Segment + RBP Segment, validates the entered DP Rate against that row's allowed band, then computes:
- **Loan Amount** = Item Price − (Item Price × DP Rate)
- **Regular EMI** = `-PMT(add-on rate ÷ 12, Term, Loan Amount)`
- **Processing Fee** = `row's fee rate × Loan Amount ÷ Term`
- **VAS Fee** (if enabled) = `9.11% × Regular EMI`
- **Total EMI** = Regular EMI + Processing Fee + VAS Fee

The rate card itself lives in the `SIL_RATE_TABLE` array in the page's script — update it there if the official rate card changes.

**Beginning Accrual** is the core concept tying tabs 2 and 3 together: when a loan's first due date isn't exactly one month after booking (e.g., due dates are fixed to a certain day of month like the 15th), the odd days between booking and the first due date accrue interest on the original loan amount. That interest is never collected inside a regular installment — it sits uncollected until the loan is fully settled, at which point it's added on top of the final payment. The cutoff is always the nearest occurrence of the due date's day-of-month that falls on or after the booking date — not necessarily the literal date entered, if that date is more than one cycle away from booking.

- **Regular installment interest**: `Opening Balance × APR/365 × days in period`
- **EMI**: `ROUND(-PMT(APR/12, Term, Loan Amount), 2)` (standard level-payment amortization)
- **Beginning Accrual**: `Loan Amount × APR/365 × (nearest due-day occurrence − Booking Date)`, added to the final installment / final payoff only

**Solar Financing** uses a related but distinct rule — a fixed **Baseline Days** cap (default 30) rather than a day-of-month match:

- **APR by term**: 24mo → 35%, 36mo → 34%, 48mo → 33% (fixed lookup, not user-entered)
- **Base EMI**: `-PMT(APR/12, Term, Loan Amount)`, plus a flat monthly Processing Fee (3% of loan ÷ term) and, if enabled, VAS/PayHinga (4% of loan ÷ term)
- **1st period interest is capped** at Baseline Days rather than the actual days from booking to the 1st due date; the excess days' interest is deferred and added to the **final** installment
- **VAS/PayHinga true-up**: every regular period charges the flat monthly VAS amount; the final period (which pays off the exact remaining balance) instead charges VAS as a percentage of that period's balance + interest
- **Effective Interest Rate (EIR)**: solved via IRR on the full cashflow series and annualized, reflecting the true cost of borrowing including fees

All formulas were validated line-for-line against the source Excel models before this tool was built.

## Notes

- All amounts are in PHP (₱).
- This is a planning/reference tool — always verify against the system of record before using for actual collections, settlements, or offers.
