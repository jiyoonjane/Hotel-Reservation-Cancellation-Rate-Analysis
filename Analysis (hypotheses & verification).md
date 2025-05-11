# 3. Hypotheses & Verification

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

