# Overpay Estimates - RoAnalytics

_Last updated: October 22, 2025_

This document explains the **system used to calculate overpay (OP) estimates** in RoAnalytics. 
It provides insight into the data patterns, value relationships, and trade dynamics behind each estimate.

> [!NOTE]
> Overpays are not manually entered. Instead, RoAnalytics uses **predictive modeling to estimate overpays automatically data or simple calculations**.

> [!CAUTION]
> Some estimates may not always be perfectly accurate. It’s always recommended to verify these overpays before trading.

RoAnalytics was built to provide reliable, data-driven insights into item information - including predicted overpays for primarily high-demand items.

---

# Table of Contents
1. [Understanding Overpays in Trading](#1-understanding-overpays-in-trading)
2. [Overpay Calculations for RAP items](#2-overpay-calculations-for-rap-items)
3. [Overpay Calculations for Valued items](#3-overpay-calculations-for-valued-items)
4. [Why Overpay is N/A](#4-why-is-the-overpay-estimate-na-not-available)
5. [Contact](#5-contact)

---

# 1. Understanding Overpays in Trading
When trading, there are two types of overpays to keep in mind: 
- **The overpay you give** - for example offering 300 overpay on a Legit
- **The overpay you recieve** - for example, recieving 600 overpay on a Legit

In an *ideal scenario*, you want to **give less overpay than what you receive**. 
Because of this, overpay values can’t be represented by a single number or range; doing so would make it unclear whether that range applies to what you should give or what you should expect to receive.

---

# 2. Overpay Calculations for RAP items
- RAP-based items rely primarily on price behavior and sale trends rather than assigned Rolimon’s values..
  - RoAnalytics estimates overpays for these items using:
    - [x] The item’s current RAP
    - [ ] *(Planned) The item’s best price*
    - [ ] *(Planned) The item’s daily sales - demand*
  
  - The general trading rule assumes an average overpay of 10% of the item’s RAP. It is then split into two separate default overpay ranges:
    - Overpay to Give:
      - The range is between 0% - 3% of the item’s RAP
    - Overpay to Recieve:
      - This range is slightly higher, at 5% - 10% of the item’s RAP
  
  - Once the default "give” and “receive” overpay ranges are calculated, **bonuses or deductions** are applied to refine the estimates based on market data:
    - Best price consideration:
      | Best Price Difference (vs RAP) | Adjustment Type | % Change Applied |
      | :----------------------------- | :-------------- | :--------------: |
      | Greater than +30%              | Bonus           |       +20%       |
      | Between +20% and +30%          | Bonus           |       +15%       |
      | Between +10% and +20%          | Bonus           |       +10%       |
      | Between 0% and +10%            | Bonus           |        +5%       |
      | Equal to RAP                   | Neutral         |        0%        |
      | Between -10% and 0%            | Deduction       |        -5%       |
      | Between -20% and -10%          | Deduction       |       -10%       |
      | Between -30% and -20%          | Deduction       |       -15%       |
      | Less than -30%                 | Deduction       |       -20%       |
    - Demand consideration:
      | Daily Sales (Average) | Adjustment Type   | % Change Applied |
      | :-------------------- | :---------------- | :--------------: |
      | More than 1.0         | Bonus             |       +15%       |
      | 0.8 – 1.0             | Bonus             |        +7%       |
      | 0.6 – 0.8             | Bonus             |        +4%       |
      | 0.5 – 0.6             | Neutral           |        0%        |
      | 0.3 – 0.5             | Deduction         |        -7%       |
      | 0.1 – 0.3             | Deduction         |        -15%      |
      | Less than 0.1         | N/A (no estimate) |         —        |
  
  - Assume the following conditions: A 4,000 R$ RAP item, with best price of 5,000 R$, and daily sales of 1.5:
    - Default Overpay Ranges:
      - Overpay to Give: 0 - 80 R$
      - Overpay to Receive: 200 - 400 R$
    - Bonuses:
      - Bonus due to best price: +15% to overpay
      - Bonus due to demand: +15% to overpay
    - Modified Overpay Ranges:
      - Overpay to Give: 0 - 100 R$ (+30%)
      - Overpay to Give: 260 - 520 R$ (+30%)
      
  - Assume the following conditions: A 6,000 R$ RAP item, with best price of 5,000 R$, and daily sales of 0.2
    - Default Overpay Ranges:
      - Overpay to Give: 0 - 120 R$
      - Overpay to Receive: 300 - 600 R$
    - Deductions:
      - Deduction due to best price: -10% to overpay
      - Deduction due to demand: -15% to overpay
    - Modified Overpay Ranges:
      - Overpay to Give: 0 - 90 R$ (-25%)
      - Overpay to Receive: 225 - 450 R$ (-25%)

---

# 3. Overpay Calculations for Valued items
- Valued Items: depend on **trading patterns, proof data, and RAP in some cases**.
  - RoAnalytics estimates overpays for these based on:
    - [x] The item’s rap to value ratio (for items less then 100,000 R$)
    - [x] A detailed prediction model, that is generated from a given set of data, which is slipt into 3 different tiers: low, mid, high
  - 

---

# 4. Why is the Overpay Estimate N/A (Not Available)?
There are several reasons why RoAnalytics may not provide an overpay estimate for a specific item:
- Item value exceeds 200K
  - As the developer, I currently focus on items valued up to 200K. Items above this range aren’t yet supported because their trade data is more complex. As I gain more experience and collect more data, this limit will gradually increase.
- Items with low or terrible demand often have unpredictable overpays, caused by market manipulation or irregular trade activity.
  - This applies for valued items with low/terrible demand tag.
    - For example, BPTH, being valued at 18,000 R$ with 40,000 R$+ proofs
  - The same logic applies to RAP items with less than 0.1 sales per day.
    - For example, Aer, with a RAP of 45,000 R$, with 40,000 R$+ proofs
- Item is projected
  - If an item is flagged as a known projected on Rolimon’s, no overpay estimate will be calculated.

Code logic used for determining eligibility:
```python
    if (
      item_value > 200000 
      or demand in ["Low", "Terrible"] 
      or daily_sales <= 0.1
      or projected
    ):
```

---

# 5. Contact
If you have questions, please contact me!  
- Discord: **luxuryrangerover**

---

Thank you for using **RoAnalytics**!  
This project is still evolving - your feedback is VERY important to improve prediction accuracy and make trading insights more reliable.
