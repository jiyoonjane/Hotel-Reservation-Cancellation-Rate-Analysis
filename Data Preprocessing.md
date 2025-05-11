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
