## Hypotheses & Verification

1. Cancellation Rate by Customer Segment

Hypothesis 1: Customers with infants have a higher cancellation rate than families or solo travelers.

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import pearsonr

# Load preprocessed data
df = pd.read_csv('hotel_bookings_cleaned.csv')

# Define segments
conditions = [
    (df['children'] > 0) | (df['babies'] > 0),
    (df['adults'] > 1) & (df['children'] + df['babies'] == 0),
    (df['adults'] == 1) & (df['children'] + df['babies'] == 0)
]
choices = ['With Baby', 'Family', 'Single']
df['segment'] = pd.Categorical(pd.np.select(conditions, choices, default='Other'),
                                 categories=choices)

# Calculate cancellation rates
rates = df[df['segment']!='Other'].groupby('segment')['is_canceled'].mean() * 100
print(rates)

# Plot
plt.figure(figsize=(6,4))
sns.barplot(x=rates.index, y=rates.values)
plt.ylabel('Cancellation Rate (%)')
plt.title('Cancellation Rate by Customer Segment')
plt.savefig('segment_cancellation_rate.png', bbox_inches='tight')

Result:

Family: ~39.1%

Single: ~29.0%

With Baby: ~18.2%

Hypothesis 1 is rejected; customers with infants cancel least.

2. Stay Length vs. Cancellation

Hypothesis 2: Longer-stay families have higher cancellation rates.

# Compute total stay length
df['stay_length'] = df['stays_in_weekend_nights'] + df['stays_in_week_nights']

# Aggregate cancellation rate by length
agg = df.groupby('stay_length')['is_canceled'].mean().reset_index()
r, p = pearsonr(agg['stay_length'], agg['is_canceled'])
print(f"Pearson r = {r:.4f}, p = {p:.4f}")

# Scatter plot
plt.figure(figsize=(6,4))
sns.scatterplot(data=agg, x='stay_length', y='is_canceled')
plt.xlabel('Stay Length (nights)')
plt.ylabel('Cancellation Rate')
plt.title('Cancellation Rate vs. Stay Length')
plt.savefig('stay_length_vs_cancellation.png', bbox_inches='tight')

Result:

Pearson r ≈ -0.3346, p < 0.05

Hypothesis 2 is only partially supported: ultra-long stays cancel more, but mid-term shows lower rates.
