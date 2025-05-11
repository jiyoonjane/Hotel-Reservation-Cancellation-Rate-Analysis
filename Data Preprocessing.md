#  Data Overview & Preprocessing

##  Dataset Summary

* **Records:** 119,390 bookings
* **Features:** 36 columns
* **Period:** July 1, 2015 – August 31, 2017
* **Location:** Two hotels in Portugal
* **Source:** Hotel Booking (Data Analysis)

### Key Columns

| Category          | Column                                                                                                                                                                            | Description                                                                              |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Basic**         | hotel, is\_canceled                                                                                                                                                               | Hotel type; cancellation flag (0 = kept, 1 = canceled)                                   |
| **Dates**         | lead\_time, arrival\_date\_year/month/week/day                                                                                                                                    | Lead time; check‑in date components                                                      |
| **Stay Details**  | stays\_in\_weekend\_nights, stays\_in\_week\_nights                                                                                                                               | Nights on weekend vs. weekday                                                            |
| **Guests**        | adults, children, babies                                                                                                                                                          | Number of travelers by age group                                                         |
| **Meals & Rooms** | meal, reserved\_room\_type, assigned\_room\_type                                                                                                                                  | Meal plan; booked vs. assigned room                                                      |
| **Booking Info**  | market\_segment, distribution\_channel, is\_repeated\_guest, previous\_cancellations, previous\_bookings\_not\_canceled, booking\_changes, deposit\_type, days\_in\_waiting\_list | Channel/segment; repeat guest flag; history of cancellations; deposit; waiting list days |
| **Payments**      | customer\_type, adr, required\_car\_parking\_spaces, total\_of\_special\_requests                                                                                                 | Customer segment; average daily rate; parking spaces; special requests                   |
| **Excluded**      | name, email, phone-number, credit\_card                                                                                                                                           | Personal identifiers (excluded from analysis)                                            |

### Missing Data

| Column   | Missing Count | Imputation       |
| -------- | ------------- | ---------------- |
| children | 4             | 0 (no children)  |
| country  | 488           | 'N/A'            |
| agent    | 16,340        | 0 (direct book)  |
| company  | 112,593       | 0 (no corporate) |

# 2. Data Overview & Preprocessing

## 2.1 Dataset Summary

* **Records:** 119,390 bookings
* **Features:** 36 columns
* **Period:** July 1, 2015 – August 31, 2017
* **Location:** Two hotels in Portugal
* **Source:** Hotel Booking (Data Analysis)

### Key Columns

| Category          | Column                                                                                                                                                                            | Description                                                                              |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Basic**         | hotel, is\_canceled                                                                                                                                                               | Hotel type; cancellation flag (0 = kept, 1 = canceled)                                   |
| **Dates**         | lead\_time, arrival\_date\_year/month/week/day                                                                                                                                    | Lead time; check‑in date components                                                      |
| **Stay Details**  | stays\_in\_weekend\_nights, stays\_in\_week\_nights                                                                                                                               | Nights on weekend vs. weekday                                                            |
| **Guests**        | adults, children, babies                                                                                                                                                          | Number of travelers by age group                                                         |
| **Meals & Rooms** | meal, reserved\_room\_type, assigned\_room\_type                                                                                                                                  | Meal plan; booked vs. assigned room                                                      |
| **Booking Info**  | market\_segment, distribution\_channel, is\_repeated\_guest, previous\_cancellations, previous\_bookings\_not\_canceled, booking\_changes, deposit\_type, days\_in\_waiting\_list | Channel/segment; repeat guest flag; history of cancellations; deposit; waiting list days |
| **Payments**      | customer\_type, adr, required\_car\_parking\_spaces, total\_of\_special\_requests                                                                                                 | Customer segment; average daily rate; parking spaces; special requests                   |
| **Excluded**      | name, email, phone-number, credit\_card                                                                                                                                           | Personal identifiers (excluded from analysis)                                            |

### Missing Data

| Column   | Missing Count | Imputation       |
| -------- | ------------- | ---------------- |
| children | 4             | 0 (no children)  |
| country  | 488           | 'N/A'            |
| agent    | 16,340        | 0 (direct book)  |
| company  | 112,593       | 0 (no corporate) |

##  Preprocessing Steps

1. **Drop PII:** Removed `name`, `email`, `phone-number`, `credit_card`.
2. **Impute Missing:** As shown above, fill child and booking‑related nulls with domain‑specific defaults.
3. **Create Derived Variables:**

   * **Categorical Encoding:** Convert `hotel`, `market_segment`, `distribution_channel`, `deposit_type`, `customer_type` to numeric codes for modeling.
   * **Family Flag (`is_family`):**

     ```python
     df['is_family'] = ((df['adults'] > 0) & ((df['children'] > 0) | (df['babies'] > 0))).astype(int)
     ```
4. **Type Conversion:** Convert `arrival_date_month` from object to integer for easier analysis.

These steps ensure a clean, model-ready dataset for subsequent EDA, statistical testing, and logistic regression modeling.

##  Preprocessing Code 

```python
import pandas as pd

# Load data
df = pd.read_csv('hotel_bookings.csv')  # replace with your file path

# 1. Drop PII columns
df = df.drop(columns=['name','email','phone-number','credit_card'], errors='ignore')

# 2. Impute missing values
df['children'] = df['children'].fillna(0)
df['country']  = df['country'].fillna('N/A')
df['agent']    = df['agent'].fillna(0)
df['company']  = df['company'].fillna(0)

# 3. Create derived variables
# 3.1 Categorical encoding
cat_cols = ['hotel','market_segment','distribution_channel','deposit_type','customer_type']
for col in cat_cols:
    df[col] = df[col].astype('category').cat.codes

# 3.2 Family flag
df['is_family'] = ((df['adults'] > 0) & ((df['children'] > 0) | (df['babies'] > 0))).astype(int)

# 4. Type conversion
# Convert arrival_date_month from string to integer (e.g. 'July'->7)
month_map = {
    'January':1,'February':2,'March':3,'April':4,'May':5,'June':6,
    'July':7,'August':8,'September':9,'October':10,'November':11,'December':12
}
df['arrival_date_month'] = df['arrival_date_month'].map(month_map).astype(int)

# Save preprocessed data
df.to_csv('hotel_bookings_cleaned.csv', index=False)
```


