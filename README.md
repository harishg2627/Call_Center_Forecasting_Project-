# Call Center Volume Forecasting & Staffing Capacity Model

## What this project does, in plain English

This project answers a question real call centers face every week: *how many calls are we
going to get, and how many agents do we need on the schedule to handle them?*

It has three parts:

1. **Forecasting** — predicting how many calls will come in on future days, using patterns
   in past data (which day of the week it is, whether it's a holiday season, an overall
   growth trend, etc.)
2. **Driver analysis** — figuring out *why* volume goes up or down on certain days, so the
   business understands what's actually causing the swings.
3. **Capacity planning** — converting the call volume forecast into a staffing number: how
   many agents are needed per day to handle the predicted volume.

## About the data

The dataset is **simulated**, not real call center data. It was generated to behave like a
real call center: more calls on weekdays than weekends, a holiday-season spike in
November/December, a dip around New Year's, a gradual upward trend as the business grows,
and random day-to-day noise. This was done because real call center data isn't publicly
available, but the patterns mimic what real workforce planning teams deal with.

**Be upfront about this in interviews.** Say something like: "I built this on a simulated
dataset designed to mimic real call center seasonality, since I didn't have access to
proprietary call center data — the same modeling approach applies directly to real volume
data." That's honest and still demonstrates the skill.

## Step 1: Understanding the data (decomposition)

Before modeling anything, the script breaks the daily call volume into three pieces using
`statsmodels.seasonal_decompose`:
- **Trend** — the slow, underlying direction (here, gradually increasing as the business grows)
- **Seasonality** — the repeating weekly pattern (busy Mon-Fri, quiet Sat-Sun)
- **Residual** — the random leftover noise that no pattern explains

This is a standard first step in any time series project — it tells you what kind of
patterns you're actually trying to model before you pick a method.

## Step 2: Two forecasting approaches

**Driver-based regression** — a linear regression model where the "drivers" are explicit
features: which day of the week it is, whether it's holiday season, whether it's the
New Year dip, and an overall trend index. This is the kind of model business stakeholders
like because each driver's effect is directly readable (see `driver_importance.png`).

**SARIMA (Seasonal ARIMA)** — a classic time series forecasting method that learns
patterns directly from the sequence of past values, including the weekly seasonal cycle,
without needing hand-built features. This is the more "pure time series" approach.

Both models were trained on all data except the last 30 days, then tested on those 30
days to see how close their predictions were to what actually happened.

**Results on the 30-day holdout:**

| Model | MAE | RMSE | MAPE |
|---|---|---|---|
| Driver-based Regression | ~53 calls | ~58 calls | ~8.5% |
| SARIMA | ~31 calls | ~38 calls | ~5.6% |

SARIMA performed better here, which makes sense — it captures the seasonal pattern more
flexibly than a fixed set of hand-built features. In a real interview, you can explain this
exact trade-off: regression is more interpretable (you can say *why* volume is higher), while
SARIMA is often more accurate but more of a "black box."

## Step 3: Turning the forecast into a staffing plan

This is the part that connects directly to actual workforce planning. Once you have a call
volume forecast, you need to know how many agents to schedule. The formula used here is a
**simplified** version of the staffing math real workforce management (WFM) teams use:

```
Required Agents = (Forecasted Calls × Average Handle Time) / (Shift Length × Target Occupancy)
```

Assumptions used: 6-minute average handle time (AHT), 8-hour shifts, 85% target occupancy
(agents aren't utilized 100% of the time — breaks, admin work, etc. eat into capacity).

**Important honesty note:** real call centers use a more sophisticated method called
**Erlang-C** (or Erlang-A), which also accounts for service-level targets and the
probability of calls queuing — this project uses a simplified version because Erlang-C
requires more advanced queuing theory math. If asked in an interview, say exactly that: you
understand this is a simplified staffing model and that production systems typically use
Erlang-C, and you'd be glad to learn/implement that with real tools like Power BI's planning
add-ins or specialized WFM software.
