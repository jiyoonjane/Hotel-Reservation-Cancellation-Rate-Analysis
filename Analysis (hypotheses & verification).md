#  Hypotheses & Verification

## 1. Cancellation Rate by Customer Segment

**Context:** A 37.04% overall cancellation rate (highlighted in red) indicates serious operational impact. We test three hypotheses by customer segment.

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import pearsonr, ttest_ind

# Load cleaned data
df = pd.read_csv('hotel_bookings_cleaned.csv')

# 1. Segment definition
conditions = [
    (df['children']>0)|(df['babies']>0),          # With Baby
    (df['adults']>1)&(df['children']+df['babies']==0),  # Family
    (df['adults']==1)&(df['children']+df['babies']==0)  # Single
]
choices = ['With Baby','Family','Single']
df['segment'] = pd.Categorical(pd.np.select(conditions,choices,default='Other'),categories=choices)

# 2. Calculate rates
data_seg = df[df['segment']!='Other']
rates = data_seg.groupby('segment')['is_canceled'].mean()*100
print(rates)

# 3. Plot
plt.figure(figsize=(6,4))
sns.barplot(x=rates.index, y=rates.values, palette='pastel')
plt.title('Cancellation Rate by Customer Segment')
plt.ylabel('Cancellation Rate (%)')
plt.savefig('figures/segment_cancellation.png',bbox_inches='tight')
```

**Results:**  Family: 39.11%, Single: 29.03%, With Baby: 18.21%.  Hypothesis 1 is rejected.

---

## 2. Stay Length vs. Cancellation

**Hypothesis 2:** Longer stay implies higher cancellation for family guests.

```python
# 1. Compute stay length
family = data_seg[data_seg['segment']=='Family'].copy()
family['stay_length'] = family['stays_in_weekend_nights'] + family['stays_in_week_nights']

# 2. Bin lengths
bins = [0,3,7,14,30,1000]
labels = ['1-3','4-7','8-14','15-30','30+']
family['length_bin'] = pd.cut(family['stay_length'],bins=bins,labels=labels,right=False)

# 3. Aggregate
bin_rates = family.groupby('length_bin')['is_canceled'].mean()*100
print(bin_rates)

# 4. Scatter & correlation
agg = family.groupby('stay_length')['is_canceled'].mean().reset_index()
r, p = pearsonr(agg['stay_length'],agg['is_canceled'])
print(f"Pearson r={r:.4f}, p={p:.4f}")

plt.figure(figsize=(6,4))
sns.scatterplot(data=agg,x='stay_length',y='is_canceled',alpha=0.6)
plt.title('Cancellation vs. Stay Length')
plt.xlabel('Stay Length (nights)')
plt.ylabel('Cancellation Rate')
plt.savefig('figures/stay_vs_cancel.png',bbox_inches='tight')
```

**Results:** Ultra-long (30+) highest rate; mid-term (8-30) lower. r≈-0.3346. Hypothesis 2 partially supported.

---

## 3. New vs. Returning & Market Segment

**Hypothesis 3:** New customers cancel more than returning.

```python
# 1. Rates by new/returning
new_rates = df.groupby('is_repeated_guest')['is_canceled'].mean()*100
print(new_rates)

plt.figure(figsize=(6,4))
sns.barplot(x=['New','Returning'],y=new_rates.values)
plt.title('New vs Returning Cancellation')
plt.ylabel('Cancellation Rate (%)')
plt.savefig('figures/new_returning.png',bbox_inches='tight')

# 2. New customers by market_segment
new = df[df['is_repeated_guest']==0]
ms_rates = new[new['market_segment']!='Undefined'].groupby('market_segment')['is_canceled'].mean()*100
ms_rates = ms_rates.sort_values(ascending=False)
print(ms_rates)

plt.figure(figsize=(8,4))
sns.barplot(x=ms_rates.index, y=ms_rates.values)
plt.xticks(rotation=45)
plt.title('New Customer Cancellation by Market Segment')
plt.ylabel('Cancellation Rate (%)')
plt.savefig('figures/new_ms_cancel.png',bbox_inches='tight')
```

**Results:** New: 37.79%, Returning: 14.49%.  Groups segment for new customers: 60.76%. Hypothesis 3 is supported.

---

# 2. Hotel Cancellation Timing Patterns

We examine two hypotheses about how cancellation rates vary by month and by weekday vs. weekend, and by hotel type.

## Hypothesis 1: Cancellation Rate Differs by Month

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import chi2_contingency

# Load raw data
raw = pd.read_csv('hotel_bookings.csv')

# Preprocess
df = raw.copy()
df['arrival_date_month'] = pd.to_datetime(df['arrival_date_month'], format='%B').dt.month

df['is_canceled'] = df['is_canceled'].astype(int)

# Overall monthly cancellation rate
monthly = df.groupby('arrival_date_month')['is_canceled'].mean()
print(monthly)

plt.figure(figsize=(8,4))
sns.barplot(x=monthly.index, y=monthly.values, palette='viridis')
plt.title('Monthly Cancellation Rate')
plt.xlabel('Month')
plt.ylabel('Cancellation Rate')
plt.savefig('figures/monthly_cancellation_rate.png', bbox_inches='tight')

# Chi-square test for uniformity
o = df['arrival_date_month'].value_counts().sort_index().values
e = [o.sum()/12]*12
chi2, p, dof, ex = chi2_contingency([o, e])
print(f"Monthly χ² = {chi2:.2f}, p = {p:.4f}")
```

**Result:** June highest (\~0.55), November lowest (\~0.20). p < 0.05 ⇒ rates differ by month.

---

## Monthly Cancellation by Hotel Type

```python
# Ensure hotel type strings
# E.g., 'City Hotel' and 'Resort Hotel'

# Group by month and hotel type
by_type = df.groupby(['hotel','arrival_date_month'])['is_canceled'].mean().reset_index()

plt.figure(figsize=(8,4))
sns.lineplot(data=by_type, x='arrival_date_month', y='is_canceled', hue='hotel', marker='o')
plt.title('Monthly Cancellation Rate by Hotel Type')
plt.xlabel('Month')
plt.ylabel('Cancellation Rate')
plt.savefig('figures/monthly_cancel_by_type.png', bbox_inches='tight')
```

## Monthly Booking Counts by Hotel Type

```python
counts = df.groupby(['hotel','arrival_date_month'])['is_canceled'].count().reset_index(name='bookings')

plt.figure(figsize=(8,4))
sns.lineplot(data=counts, x='arrival_date_month', y='bookings', hue='hotel', marker='s')
plt.title('Monthly Booking Counts by Hotel Type')
plt.xlabel('Month')
plt.ylabel('Number of Bookings')
plt.savefig('figures/monthly_counts_by_type.png', bbox_inches='tight')
```

## Family Proportion by Hotel Type

```python
# Define family bookings
df['is_family'] = ((df['adults']>1)&((df['children']>0)|(df['babies']>0)))

# Overall proportion
overall = df.groupby('hotel')['is_family'].mean()
print(overall)

# Summer months only (7,8)
summer = df[df['arrival_date_month'].isin([7,8])]
sum_prop = summer.groupby('hotel')['is_family'].mean()
print(sum_prop)
```

**Observation:** Resort hotels have higher overall and summer family booking ratios.

---

## Hypothesis 2: Weekend vs. Weekday Cancellations

```python
# Define weekend bookings: any weekend nights>0
df['is_weekend'] = df['stays_in_weekend_nights'] > 0

# Compute cancellation rates
t1 = df.groupby('hotel')['is_canceled'].mean()
t2 = df[df['is_weekend']].groupby('hotel')['is_canceled'].mean()
t3 = df[~df['is_weekend']].groupby('hotel')['is_canceled'].mean()

print("Overall by hotel:", t1)
print("Weekend by hotel:", t2)
print("Weekday by hotel:", t3)

# Bar plot comparison
rates_df = pd.DataFrame({
    'Weekend': t2,
    'Weekday': t3
}).reset_index().melt(id_vars='hotel', var_name='Period', value_name='CancelRate')

plt.figure(figsize=(8,4))
sns.barplot(data=rates_df, x='hotel', y='CancelRate', hue='Period')
plt.title('Weekend vs. Weekday Cancellation Rate by Hotel Type')
plt.ylabel('Cancellation Rate')
plt.savefig('figures/weekend_weekday_cancel.png', bbox_inches='tight')
```

**Result:** City hotels show slightly higher weekday cancellations; resort hotels show higher weekend cancellations.

---
# 3. Booking-Level Cancellation Patterns

We analyze how deposit policy, market segment, and lead time anomalies relate to cancellation behavior.

## Cancellation Rate by Deposit Type

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Load cleaned data
df = pd.read_csv('hotel_bookings_cleaned.csv')

# 1. Cancellation rate by deposit_type
cancel_rate_deposit = df.groupby('deposit_type')['is_canceled'].mean() * 100
print(cancel_rate_deposit)

# 2. Plot
plt.figure(figsize=(6,4))
sns.barplot(x=cancel_rate_deposit.index, y=cancel_rate_deposit.values, palette='cool')
plt.ylabel('Cancellation Rate (%)')
plt.title('Cancellation Rate by Deposit Type')
plt.savefig('figures/cancel_by_deposit.png', bbox_inches='tight')
```

**Key Insight:** Non Refundable bookings show \~99.4% cancellation despite being a small share.

---

## Booking Changes by Deposit Type

```python
# Distribution of booking_changes by deposit_type
changes_dist = (
    df.groupby('deposit_type')['booking_changes']
      .value_counts(normalize=True)
      .rename('proportion')
      .reset_index()
)
print(changes_dist)

# Heatmap
pivot_changes = changes_dist.pivot('booking_changes','deposit_type','proportion')
plt.figure(figsize=(6,4))
sns.heatmap(pivot_changes, annot=True, fmt='.2f')
plt.title('Proportion of Booking Changes by Deposit Type')
plt.xlabel('Deposit Type')
plt.ylabel('Number of Changes')
plt.savefig('figures/changes_by_deposit.png', bbox_inches='tight')
```

**Observation:** \~96% of Non Refundable & Group bookings have zero changes, linking strict policies to high cancellation.

---

## Lead Time Outliers by Deposit Type

```python
# Identify IQR bounds
o = df['lead_time']
Q1 = o.quantile(0.25)
Q3 = o.quantile(0.75)
IQR = Q3 - Q1
lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

# Count outliers per deposit_type
outlier_counts = (
    df.assign(is_outlier = (df['lead_time'] < lower) | (df['lead_time'] > upper))
      .groupby('deposit_type')['is_outlier']
      .sum()
)
print(outlier_counts)

# Boxplot
df_filtered = df[df['lead_time'] <= upper]
plt.figure(figsize=(6,4))
sns.boxplot(x='deposit_type', y='lead_time', data=df_filtered)
plt.title('Lead Time Distribution by Deposit Type (Outliers Removed)')
plt.ylabel('Lead Time (days)')
plt.savefig('figures/leadtime_by_deposit.png', bbox_inches='tight')
```

**Insight:** All deposit types exhibit long-tail lead times, with Non Refundable and No Deposit showing more extreme values.

---

## Market Segment Cancellation Rates

```python
# Cancellation rate by market_segment
ms_cancel = df[df['market_segment']!='Undefined']
                .groupby('market_segment')['is_canceled']
                .mean() * 100
print(ms_cancel.sort_values(ascending=False))

# Plot
plt.figure(figsize=(8,4))
sns.barplot(x=ms_cancel.index, y=ms_cancel.values)
plt.xticks(rotation=45)
plt.ylabel('Cancellation Rate (%)')
plt.title('Cancellation Rate by Market Segment')
plt.savefig('figures/cancel_by_segment.png', bbox_inches='tight')
```

**Key Finding:** 'Groups' segment shows \~61% cancellation, compounded by strict deposit policies.

---

# 4. Lead Time vs. Cancellation Analysis

We test whether longer lead times (days between booking and check-in) increase cancellation probability.

## Hypotheses

* **H₀:** Lead time is unrelated to cancellation.
* **H₁:** Longer lead times have higher cancellation rates.

## Analysis Steps & Code

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import skew, kurtosis, mannwhitneyu
from scipy.stats import gaussian_kde

# 1. Load data
df = pd.read_csv('hotel_bookings_cleaned.csv')

# 2. Descriptive stats for lead_time
tl = df['lead_time']
print(f"Skewness = {skew(tl):.2f}")
print(f"Kurtosis = {kurtosis(tl):.2f}")

# 3. Mann-Whitney U test
d1 = df[df['is_canceled']==1]['lead_time']
d0 = df[df['is_canceled']==0]['lead_time']
u_stat, p_val = mannwhitneyu(d1, d0, alternative='two-sided')
print(f"Mann-Whitney U = {u_stat:.1f}, p = {p_val:.1e}")
print(f"Median canceled = {d1.median():.0f} days, median kept = {d0.median():.0f} days")

# 4. Remove outliers via IQR for plotting
Q1 = tl.quantile(0.25)
Q3 = tl.quantile(0.75)
IQR = Q3 - Q1
lower = Q1 - 1.5*IQR
upper = Q3 + 1.5*IQR
df_trim = df[(tl >= lower) & (tl <= upper)]

# 5. Boxplot of lead_time by cancellation
df_trim['status'] = df_trim['is_canceled'].map({0:'Kept',1:'Canceled'})
plt.figure(figsize=(6,4))
sns.boxplot(x='status', y='lead_time', data=df_trim)
plt.title('Lead Time by Cancellation Status (Outliers Removed)')
plt.ylabel('Lead Time (days)')
plt.savefig('figures/leadtime_boxplot.png', bbox_inches='tight')

# 6. KDE plot and find intersection point
x = np.linspace(0, df['lead_time'].max(), 1000)
kde_c = gaussian_kde(d1)
kde_k = gaussian_kde(d0)
y_c = kde_c(x)
y_k = kde_k(x)
# Find approximate crossover
diff = y_c - y_k
idx = np.where(np.sign(diff[:-1]) != np.sign(diff[1:]))[0]
cross = x[idx[0]] if idx.size else np.nan

plt.figure(figsize=(6,4))
sns.kdeplot(d0, label='Kept', shade=True)
sns.kdeplot(d1, label='Canceled', shade=True)
if not np.isnan(cross):
    plt.axvline(cross, color='black', linestyle='--',
                label=f'Cross @ {cross:.1f} days')
plt.title('Lead Time KDE by Cancellation Status')
plt.xlabel('Lead Time (days)')
plt.ylabel('Density')
plt.legend()
plt.savefig('figures/leadtime_kde.png', bbox_inches='tight')
```

## Key Results

* **Skewness:** +1.35 (right-skewed)
* **Kurtosis:** 1.70 (platykurtic)
* **Mann-Whitney U:** \~2.29e9, **p < 0.001**; medians: 113 days (canceled) vs. 45 days (kept)
* **Crossover Point:** ≈44.4 days where cancellation density equals retention

**Conclusion:** Longer lead times significantly increase cancellation risk.
**Actionable Insight:** Target bookings >44 days lead time with prepayment incentives, reminder messages, or discounts to reduce cancellations.



