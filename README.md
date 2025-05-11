
#  Introduction

##  Topic Selection & Project Objectives

### Background

* Reservation cancellation rates directly impact a hotel’s profitability and operational efficiency.
* High cancellation rates make it difficult to forecast room availability, creating challenges for operations teams.
* Analyzing factors such as customer segment, booking channel, seasonality, and lead time can uncover drivers of cancellations and inform strategies to minimize them.

### Narrative Scenario

> The hotel operations team is struggling with unpredictable room availability and revenue forecasting due to high cancellation rates. What strategies can reduce cancellations and stabilize operations?

### Project Goals

This project leverages data visualization and analysis in Python to uncover cancellation patterns and recommend optimization strategies. Specific objectives include:

1. Analyze cancellation patterns by customer segment (e.g., corporate vs. leisure).
2. Examine seasonal trends in cancellation rates.
3. Investigate booking-level factors influencing cancellations.
4. Explore the relationship between lead time (booking to check-in) and cancellation likelihood.
5. Develop and evaluate a logistic regression model to predict cancellations.

##  Research Methods & Expected Outcomes

* **Exploratory Data Analysis (EDA):** Use Python libraries (Pandas, Matplotlib, Seaborn) to visualize cancellation patterns and identify key trends.
* **Statistical Modeling:** Build a logistic regression model to quantify the impact of each factor on cancellation probability.
* **Optimization Strategy:** Identify and prioritize interventions based on model insights (e.g., targeted communication strategies for high-risk bookings).

**Expected Impact:**

* Provide data-driven recommendations to reduce cancellation rates.
* Enhance room occupancy forecasting and revenue management.
* Offer actionable insights that hotel operations can implement to improve profitability and guest satisfaction.




#  Analysis Results & Expected Impact

## 1. Cancellation Rate by Customer Segment

* **Overall Cancellation Rate:** 37.0% of all bookings
* **With Infants:** 18.2% (lower than families at 39.1% and solo travelers at 29.0%)
* **Long-Stay Families (≥30 days):** Highest cancellations; mid‑term stays (8–30 days) showed lower rates
* **New vs. Returning Guests:** New customers cancel at 37.8% vs. returning at 14.5%; packages (Groups) by new guests cancel at 60.8%

**Expected Actions:**

* Enforce stricter deposit/refund policies for high‑risk segments (new customers, package groups).
* Add confirmation steps or special incentives for ultra-long family stays.

---

## 2. Booking-Level Cancellation Patterns

* **Deposit Types:** 87% no-deposit, 12% non‑refundable
* **Non‑Refundable Cancellations:** 99.4% cancellation rate
* **Group Bookings:** 32% choose non‑refundable option, of which 75% still cancel
* **Amendments:** Zero changes common in group & non‑refundable bookings → inability to modify leads to cancellations
* **Lead Time:** Longer lead times correlate with higher cancellation rates

**Expected Actions:**

* Replace non‑refundable deposits with higher flexible rates to reduce friction.
* Allow at least one amendment for group/TA bookings.
* Introduce tiered change fees based on lead time.

---

## 3. Seasonal & Hotel-Type Cancellation Trends

* **Resort Hotels:** Peak cancellations in summer (family vacation season)
* **City Hotels:** Steady cancellations year‑round with business‑week peaks in spring/fall

**Expected Actions:**

* City hotels: mid‑week promotions and flexible cancellation terms.
* Resorts: early‑bird family packages and summer‑season incentives.

---

## 4. Lead Time vs. Cancellation Rate

* **Median Lead Time:** 113 days (canceled) vs. 45 days (kept)
* **Stat Test:** Mann-Whitney U p < 0.001 confirms longer lead times drive cancellations
* **Threshold:** Cancellations spike beyond \~44 days lead time

**Expected Actions:**

* Target long‑lead bookings (>44 days) with prepayment discounts, reminder messages, and check‑in incentives.
* Segment marketing by lead‑time cohort.

---

## 5. Logistic Regression Insights

* **High-Risk Groups:**

  * Transient (solo) bookings: ×4.24 odds of cancellation
  * TA/TO channel: ×2.18 odds
  * Repeat cancellers: ×5.46 odds
* **Protective Factors:**

  * Multiple amendments: 37% lower odds
  * Special requests: 51% lower odds
  * Parking request: 99.98% lower odds
* **Hotel Type:**

  * City hotels: higher cancellation risk vs. resorts

**Expected Actions:**

* Incentivize direct and refundable bookings (discounts, amenities).
* Implement cancellation‑score policies for habitual cancellers.
* Encourage bookings with special requests and flexible amendments.

---

##  Key Takeaways & Next Steps

* **Policy Adjustments:** Revise deposit/refund rules and amendment policies by segment and lead time.
* **Tailored Promotions:** Develop targeted offers for high-risk cohorts (new guests, long‑lead, solo business travelers).
* **Data Gaps:** Incorporate real-time booking behavior, revenue impact, and guest feedback for ongoing refinement.

