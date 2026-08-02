# Fuel Market Insights — Power BI Dashboard

A Power BI project I built to make sense of 23 years of oil, gas and fuel futures data — where the market moved, when it broke, and what that means if you're trying to trade or plan around it.

## Why I built this

Futures data on oil and gas is everywhere, but most of it just sits in spreadsheets. Nobody actually looks at it until something goes wrong — a price spike, a crash, a pandemic. I wanted to build something that lets you *see* the market move: when open positions surged, when volatility spiked, and how the five major fuel types (Crude Oil, Natural Gas, Heating Oil, RBOB Gasoline, Brent Crude Oil) stack up against each other.

So instead of a report full of numbers, I built a 6-page interactive dashboard that answers the questions a trader, analyst, or researcher would actually ask: *is this a good time to be in the market? which fuel is the safest bet? what happened in 2008, and again in 2020, and why?*
fuel-market-insights/
│
├── README.md
├── dashboard_powerbi.pbix → the full Power BI file, open it in Power BI Desktop
├── report.pdf → the written business intelligence report
│
└── images/
├── 01_home.png
├── 02_annual_market_overview.png
├── 03_market_fluctuation_insights.png
├── 04_yearly_matrix_position_overview.png
├── 05_dynamic_market_analytics.png
└── 06_fuel_type_market_overview.png


---

## The dashboard

### Home
Every good dashboard needs a front door. This is mine — a simple landing page that routes to each of the five analysis views.

![Home page](images/01_home.png)

### Annual Market Overview
This is the page I keep coming back to. Four charts, one story: the market was basically flat and quiet until 2005, then it woke up.

![Annual Market Overview](images/02_annual_market_overview.png)

- **Total open positions climbed from under 1bn in 2000 to a peak of 54bn in 2018.** That's not gradual growth — it's a market that fundamentally changed scale.
- **2008 stands out as the most volatile year on record**, with a daily price change of –556 and average close price swinging up to 63 before collapsing. That's the global financial crisis showing up directly in the data.
- **2020 is the other obvious scar** — volume spiked toward 13.7M and the average close price dropped as low as –2 in places, which lines up exactly with the pandemic disruption to fuel demand.
- Trading volume has a clear long-term upward trend, from under 0.1M in the early 2000s to consistently multi-million-unit days by the 2020s.

### Market Fluctuation Insights
This page is about volatility specifically — not just "did the price move" but "how violently."

![Market Fluctuation Insights](images/03_market_fluctuation_insights.png)

- **2022 had the single most extreme swing in the whole dataset:** +75% positive change followed almost immediately by a –74% negative change in the same window. That kind of whiplash is exactly the sort of thing a risk manager needs to see coming.
- The 7-day rolling average close price tells a smoother story: a steady climb from 9 in 2000 to a peak of 44 around 2013, a pullback through 2015–2020, then a recovery back up to 32 by 2023.
- High and low prices track each other almost perfectly across the years — which makes sense for futures contracts, but it's still satisfying to see it confirmed rather than assumed.

### Yearly Matrix Position Overview
The numbers-first page. If you just want the totals without wading through charts, this is it.

![Yearly Matrix Position Overview](images/04_yearly_matrix_position_overview.png)

- **Total open positions: 615.25bn**
- **Total close positions: 614.99bn**
- **Total volume: 14bn**
- 2008 alone accounts for roughly 7% of total open and close positions across the full 23-year period — a single year carrying an outsized share of all trading activity in the dataset.

### Dynamic Market Analytics
This is the "drill into a specific window" page — four KPI cards plus two charts that update live as you move the date slicer.

![Dynamic Market Analytics](images/05_dynamic_market_analytics.png)

- **Average daily close price across the full period: 27.92**
- **7-day rolling average: 32.32**
- **Price volatility index: 0.41**
- **Max single-day volume: 2M**
- The annual daily price change chart makes the crisis years jump out immediately — 2008 (–556), 2015 (–462), 2020 (–284) are all clearly the worst years to have been holding a long position, while 2013 (+140) and 2021 (+292) were the strongest recovery years.

### Fuel Type Market Overview
And finally — how do the five fuel types actually compare to each other?

![Fuel Type Market Overview](images/06_fuel_type_market_overview.png)

- **Crude Oil, Natural Gas, Heating Oil and RBOB Gasoline are remarkably close to each other** across open, high, low and close prices — all sitting within about a percentage point of one another (roughly 20.5% share each).
- **Brent Crude Oil is the clear outlier, consistently 2–3 percentage points lower** across every single metric — open, high, low, close, and volume. It's the smallest, least active of the five markets in this dataset.
- Volume tells the same story: Crude Oil, Natural Gas, Heating Oil and RBOB Gasoline are all sitting around 20.4–20.5% of total volume, while Brent Crude trails at 18.1%.

---

## How I built it

**Data prep (Power Query):** the raw data came in as five separate tables, one per fuel type. I appended them into a single table using `Table.Combine`, then used a `Table.NestedJoin` to merge the fuel type label back in — so every row is tagged by which fuel it belongs to, which is what makes the fuel-type comparison page possible.

**Data model:** a straightforward star-schema-style setup, with the combined fuel data as the fact table and the individual fuel tables feeding into it.

**DAX:** most of the interesting logic lives here. A few worth calling out:

```DAX
Rolling Average Close Price =
VAR Numdays = 7
VAR RollingAverage =
    CALCULATE(
        AVERAGE('Append'[close]),
        DATESINPERIOD('Append'[date], LASTDATE('Append'[date]), -Numdays, DAY)
    )
RETURN RollingAverage
```

```DAX
Price Volatility =
AVERAGEX('Append', ABS('Append'[close] - 'Append'[open]))
```

```DAX
PercentagePositiveChange =
DIVIDE([PositivePriceChanges], [TotalPriceChanges], 0)
```

The full set of measures — total open/close positions, daily price change, positive/negative change splits, and the date-part helpers (Year, Month, Quarter, Day) — is documented with explanations in `report.pdf`.

**Design:** dark maroon and amber theme, built to look like a trading terminal rather than a corporate slide deck. Every page has a date slicer (23-08-2000 to 29-12-2023) so you can zoom into any window you care about.

---

## What I'd tell someone using this data to trade or plan around it

1. **Crude Oil dominates.** If you're only going to track one fuel type closely, this is the one — it consistently leads on volume, open price, and high value.
2. **Watch for the pattern, not just the number.** 2008 and 2020 both show the same signature: a volume spike paired with a sharp negative price change. That combination is a useful early-warning signal, not just a historical curiosity.
3. **Diversify across fuel types where you can.** Crude Oil, Natural Gas, Heating Oil and RBOB Gasoline move closely together, so spreading across just those four doesn't buy you much protection — Brent Crude Oil is genuinely the odd one out and worth holding as a hedge.
4. **The 7-day rolling average smooths out noise better than the raw daily close.** If you're building any kind of trend-following logic on top of this data, start there.

---


## Full write-up

The complete business intelligence report — methodology, every chart broken down with figures, the DAX and M query documentation, and the self-assessment — is in `report.pdf`.
**Author:** Hemanth Kumar Reddy Bogathi
**Dataset:** [Oil, Gas and Other Fuels Future Data](https://www.kaggle.com/) (Kaggle).
**Tools:** Power BI Desktop · DAX · Power Query (M)

---
