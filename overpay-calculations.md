# Overpay Estimates - RoAnalytics

_Last updated: November 7, 2025_

This document explains the **system used to calculate overpay (OP) estimates** in RoAnalytics. 
It provides insight into the data patterns, value relationships, and trade dynamics behind each estimate.

> **💡 NOTE:** Overpays are not manually entered. Instead, RoAnalytics uses **predictive modeling** to estimate overpays automatically.

---

# 1. Overpay Calculations for RAP items
- **RAP-based items rely primarily on price behavior and sale trends rather than assigned Rolimon’s values.**
  - RoAnalytics estimates the overpay ranges for these items using three key factors:
    1. The item’s current RAP
    2. The item’s best price
    3. The item’s daily sales -> Measures demand
  
  1. A general trading assumption is that these items trade for about 10% of their RAP in overpay. This 10% baseline is then divided into two default ranges:
     | Overpay Type  | Range                       |
     | :----         | :--------------------       |
     | Give          | 0% - 3% of the item’s RAP   |
     | Recieve       | 5% - 10% of the item’s RAP  |
  
  2. Once the base overpay ranges are established, adjustments (bonus/drop) are applied based on best price (first table) and demand (second table). The % will shift the whole range of that item's overpay, preserving width, more on it later.
     | **BP vs RAP Difference**       | **Adjustment**  | **% Change**     |
     | :----------------------------- | :-------------- | :--------------: |
     | > +30%                         | Bonus           |       +15%       |
     | +20% to +30%                   | Bonus           |       +10%       |
     |  +10% to +20%                  | Bonus           |      +7.5%       |
     |  0% to +10%                    | Bonus           |        +5%       |
     | =                              | Neutral         |         0%       |
     |  -10% to 0%                    | Drop            |       -15%       |
     |  -20% to -10%                  | Drop            |       -20%       |
     |  -30% to -20%                  | Drop            |       -25%       |
     | < -30%                         | Drop            |       -30%       | 

     | **Daily Sales**       | **Adjustment**    | **% Change**     |
     | :-------------------- | :---------------- | :--------------: |
     | > 1.0                 | Bonus             |        +15%      |
     | 0.8 – 1.0             | Bonus             |        +10%      |
     | 0.6 – 0.8             | Bonus             |         +5%      |
     | 0.5 – 0.6             | Neutral           |          0%      |
     | 0.3 – 0.5             | Drop              |        -15%      |
     | 0.1 – 0.3             | Drop              |        -20%      |
     | < 0.1                 | -                 |          -       | 
  - Examples:
    - 4,000 R$ RAP item, with best price of 5,000 R$, and daily sales of 1.5:
      1. Default Overpay Ranges:
        - Overpay to Give: 0 - 120 R$
        - Overpay to Receive: 200 - 400 R$
      2. Calculate Multipliers:
        - Best Price vs RAP: 
          - 5,000 / 4,000 -> bp_diff = +25% -> bp_multiplier = 1.10
        - Daily Sales:
          - 1.5 -> demand_multiplier = 1.15
        - Total multiplier = 1.10 × 1.15 ~= 1.265
      3. Shift the Whole Range:
        - Give Range Midpoint = (0 + 120)/2 = 60
        - Half-width = (120 – 0)/2 = 60
        - New midpoint = 60 × 1.265 ≈ 75.9
        - Final Give Low = 75.9 – 60 ≈ 16
        - Final Give High = 75.9 + 60 ≈ 136
          - Final Give: 16 – 136 R$
        - Receive Range Midpoint = (200 + 400)/2 = 300
        - Half-width = (400 – 200)/2 = 100
        - New midpoint = 300 × 1.265 ≈ 379.5
        - Final Receive Low = 379.5 – 100 ≈ 280
        - Final Receive High = 379.5 + 100 ≈ 480
          - Final Receive: 280 – 480 R$
      4. Percent Change:
        - Total % change = (1.265 – 1) × 100 ≈ +27%
      5. Final Overpay Ranges:
        - Overpay to Give: 16 – 136	+27%
        - Overpay to Receive: 280 – 480	+27%

---

# 2. Overpay Calculations for Valued items
- Valued Items: depend on **trading patterns, proof data, and RAP in some cases**.
  - RoAnalytics estimates overpays for these based on:
    - A detailed prediction model, that is generated from a given set of data, which is slipt into 3 different tiers: low, mid, high
    - The item’s rap to value ratio (for items less then 100,000 R$)
- The system relies on Python-based regression models that learn the relationship between an item’s assigned value and its observed overpay ranges across multiple tiers (low, mid, high)

- **How the Model Works**
  1. **Dataset Preparation**
      - RoAnalytics maintains three internal datasets:
        - data_low — for low-tier items
        - data_mid — for mid-tier items
        - data_high — for high-tier items
      
      - Each dataset includes the item’s name, its assigned Rolimon’s value, and four key metrics that define the ranges for the overpays:
        - Overpay_To_Give_Lower
        - Overpay_To_Give_Upper
        - Overpay_To_Receive_Lower
        - Overpay_To_Receive_Upper
      - The data set used by RoAnalytics doesn’t need to be updated whenever item values change - overpay trends remain consistent enough for the models to stay accurate over time.
  2. **Model Training**
       - The system uses scikit-learn to train a separate predictive model for each of the four overpay metrics.
       - A Ridge regression model (a regularized form of linear regression) is applied (degree is always 1 (linear) and alpha is 10).
       - The following code snippit looks like this (X is the item's tier (low, mid high) y is the item's value):
         ```python
         def make_model(y):
          model = make_pipeline(PolynomialFeatures(1), Ridge(alpha=10))
          model.fit(X, y)
          return model
         ```
  3. **Prediction**
       - Once trained, the models can estimate overpay ranges for any item within the range of the provided data.
  4. **Model Tiering**
       - Separate models are maintained for different market ranges to ensure better accuracy:
            | **Tier**  | **Value Range**           | **Model Used**   |
           | :---- | :-------------------- | :----------  |
           | Low   | <= 11,000 R$          | `low_models` |
           | Mid   | 12,000 – 80,000 R$    | `mid_models` |
           | High  | 80,000 – 200,000 R$   | `high_models`| 
  5. **Continuous Improvement**
       - As more trading data and community observations are collected, these datasets are expanded - retraining the models to make overpay predictions even more accurate over time.
  6. **The Data**
   ```python
    data_low = [
        {"Item": "Legit", "Value": 4500, "Overpay_To_Give_Lower": 200, "Overpay_To_Give_Upper": 300, "Overpay_To_Receive_Lower": 500, "Overpay_To_Receive_Upper": 700},
        {"Item": "GCWHP", "Value": 6000, "Overpay_To_Give_Lower": 400, "Overpay_To_Give_Upper": 500, "Overpay_To_Receive_Lower": 600, "Overpay_To_Receive_Upper": 900},
        {"Item": "STA", "Value": 7000, "Overpay_To_Give_Lower": 550, "Overpay_To_Give_Upper": 750, "Overpay_To_Receive_Lower": 900, "Overpay_To_Receive_Upper": 1100},
        {"Item": "BTH", "Value": 9000, "Overpay_To_Give_Lower": 700, "Overpay_To_Give_Upper": 900, "Overpay_To_Receive_Lower": 1000, "Overpay_To_Receive_Upper": 1300}
    ]

    data_mid = [
        {"Item": "BIA", "Value": 12000, "Overpay_To_Give_Lower": 800, "Overpay_To_Give_Upper": 1000, "Overpay_To_Receive_Lower": 1200, "Overpay_To_Receive_Upper": 1500},
        {"Item": "GWW", "Value": 14000, "Overpay_To_Give_Lower": 900, "Overpay_To_Give_Upper": 1100, "Overpay_To_Receive_Lower": 1400, "Overpay_To_Receive_Upper": 1600},
        {"Item": "Undead", "Value": 18000, "Overpay_To_Give_Lower": 1200, "Overpay_To_Give_Upper": 1500, "Overpay_To_Receive_Lower": 1800, "Overpay_To_Receive_Upper": 2300},
        {"Item": "8BRC", "Value": 20000, "Overpay_To_Give_Lower": 1500, "Overpay_To_Give_Upper": 1800, "Overpay_To_Receive_Lower": 2000, "Overpay_To_Receive_Upper": 2600},
        {"Item": "CS", "Value": 28000, "Overpay_To_Give_Lower": 2000, "Overpay_To_Give_Upper": 2300, "Overpay_To_Receive_Lower": 2500, "Overpay_To_Receive_Upper": 3300},
        {"Item": "Dupa", "Value": 38000, "Overpay_To_Give_Lower": 2500, "Overpay_To_Give_Upper": 3000, "Overpay_To_Receive_Lower": 3500, "Overpay_To_Receive_Upper": 4500},
        {"Item": "Supa", "Value": 55000, "Overpay_To_Give_Lower": 3500, "Overpay_To_Give_Upper": 5000, "Overpay_To_Receive_Lower": 5500, "Overpay_To_Receive_Upper": 6500},
        {"Item": "RBM", "Value": 65000, "Overpay_To_Give_Lower": 4000, "Overpay_To_Give_Upper": 6000, "Overpay_To_Receive_Lower": 6500, "Overpay_To_Receive_Upper": 7500},
        {"Item": "GQOTN", "Value": 80000, "Overpay_To_Give_Lower": 5000, "Overpay_To_Give_Upper": 7000, "Overpay_To_Receive_Lower": 8000, "Overpay_To_Receive_Upper": 9000}
    ]
  
    data_high = [
        {"Item": "Madness", "Value": 90000, "Overpay_To_Give_Lower": 7000, "Overpay_To_Give_Upper": 8000, "Overpay_To_Receive_Lower": 9000, "Overpay_To_Receive_Upper": 10000},
        {"Item": "PV", "Value": 105000, "Overpay_To_Give_Lower": 7000, "Overpay_To_Give_Upper": 9000, "Overpay_To_Receive_Lower": 10500, "Overpay_To_Receive_Upper": 12000},
        {"Item": "BBM", "Value": 115000, "Overpay_To_Give_Lower": 8500, "Overpay_To_Give_Upper": 10000, "Overpay_To_Receive_Lower": 11500, "Overpay_To_Receive_Upper": 13000},
        {"Item": "BM", "Value": 125000, "Overpay_To_Give_Lower": 9000, "Overpay_To_Give_Upper": 11000, "Overpay_To_Receive_Lower": 12500, "Overpay_To_Receive_Upper": 14000},
        {"Item": "Scissors", "Value": 135000, "Overpay_To_Give_Lower": 10000, "Overpay_To_Give_Upper": 11500, "Overpay_To_Receive_Lower": 13500, "Overpay_To_Receive_Upper": 15000},
        {"Item": "TM", "Value": 150000, "Overpay_To_Give_Lower": 11000, "Overpay_To_Give_Upper": 13000, "Overpay_To_Receive_Lower": 15000, "Overpay_To_Receive_Upper": 17000},
        {"Item": "BIH", "Value": 170000, "Overpay_To_Give_Lower": 13000, "Overpay_To_Give_Upper": 15000, "Overpay_To_Receive_Lower": 17000, "Overpay_To_Receive_Upper": 20000},
        {"Item": "SEOTN", "Value": 190000, "Overpay_To_Give_Lower": 15000, "Overpay_To_Give_Upper": 17000, "Overpay_To_Receive_Lower": 19000, "Overpay_To_Receive_Upper": 24000}
    ]
   ```
   
- **RAP-to-Value Ratio Adjustment (for items less then 100,000 R$)**
  - RoAnalytics also accounts for the RAP-to-Value ratio (RAP divided by value) of each limited item when generating its overpay range estimates. **The ratio influences whenever a valued item will recieve a bonus or deduction.**
  - This ratio provides insight into how stable an item’s RAP is relative to its assigned value.
      | **Tier**    | **Ratio**   | **Adjustment** | **% Change** |
      | :------ | :----------------   | :-------------- | :--------------: |
      | Low | >= 0.97             | Bonus           |       +15%       |
      | Low | <= 0.85             | Deduction       |       -15%       |
      | Mid | >= 0.99             | Bonus           |       +15%       |
      | Mid | <= 0.90             | Deduction       |       -15%       | 
    - Anything between does not contain a bonus, or deduction to the item's overpay (0%).
---

# 3. Why is the Overpay Estimate N/A (Not Available)?
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

---

# 4. Contact
If you have questions, please contact me!  
- Discord: **luxuryrangerover**
