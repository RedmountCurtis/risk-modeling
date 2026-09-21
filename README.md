# Adding tickers to the Security Library (Supabase SQL)

The Redmount tool's Security Library (risk scores + expense ratios per ticker) is stored in Supabase, in the `risk_settings` table, `security_library` column (JSONB array). The app loads this on login and it overrides whatever default list is baked into `index.html`, so adding a ticker to the code alone does not update a live account — you need to get it into this Supabase column.

## Fastest way: SQL Editor

Run this in the Supabase SQL Editor for the Redmount project. It **appends** new tickers without touching existing ones (safe to re-run as long as the tickers being added are genuinely new):

```sql
update risk_settings
set security_library = security_library || '[
  {"ticker":"TICKER","name":"Full Name","assetClass":"us_large","score":50,"expenseRatio":0.0}
]'::jsonb;
```

Each entry needs:
- `ticker` — uppercase symbol
- `name` — display name
- `assetClass` — one of: `us_large`, `us_small`, `us_reit`, `intl_large`, `em_equity`, `balanced`, `us_bonds`, `us_hy`, `us_tips`, `us_int_treasury`, `us_long_treasury`, `intl_govt_bonds`, `em_sovereign`, `private_equity`, `private_re`, `infrastructure`, `commodities`, `us_cash`, `lev`
- `score` — 0–100 risk score (advisor's own judgment call, not sourced from a live feed)
- `expenseRatio` — as a percent, e.g. `0.42` for 0.42%

## Alternative: in-app UI

The tool also has a "Security Library" view with a **Bulk add/update** paste box — rows in the form `TICKER,Name,AssetClassKey,Score,ExpenseRatio`, one per line. Existing tickers get updated in place, new ones get added. This saves to Supabase automatically, same effect as the SQL above.

## Log of additions

**2026-09-21** — Added 6 tickers found in "My Saved Portfolios.xlsx" that weren't yet in the library:

```sql
update risk_settings
set security_library = security_library || '[
  {"ticker":"POWR","name":"iShares U.S. Power Infrastructure ETF","assetClass":"us_large","score":52,"expenseRatio":0.42},
  {"ticker":"FLOT","name":"iShares Floating Rate Bond ETF","assetClass":"us_cash","score":7,"expenseRatio":0.15},
  {"ticker":"ICVT","name":"iShares Convertible Bond ETF","assetClass":"balanced","score":50,"expenseRatio":0.35},
  {"ticker":"CASH-USD","name":"Cash (USD)","assetClass":"us_cash","score":0,"expenseRatio":0},
  {"ticker":"IALT","name":"iShares Systematic Alternatives Active ETF","assetClass":"balanced","score":45,"expenseRatio":0.5},
  {"ticker":"NEAR","name":"iShares Short Maturity Bond ETF","assetClass":"us_cash","score":7,"expenseRatio":0.25}
]'::jsonb;
```

(These were also added to the hardcoded default list in `index.html` itself, so a brand-new/never-logged-in account would have them too — but for an existing account, the SQL above is what actually takes effect.)
