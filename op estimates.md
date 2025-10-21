# Overpay Estimates — RoAnalytics

_Last updated: October 12, 2025_

This document explains the **methodology used to calculate overpay (OP) estimates** in RoAnalytics. 
It provides insight into the data patterns, value relationships, and trade dynamics behind each estimate.

### Keep in mind that overpays are not manually entered - doing so would take an enormous amount of time and effort. 
Instead, RoAnalytics uses **predictive modeling to estimate overpays automatically data and calculations**. 
Because of this, some estimates may not always be perfectly accurate. It’s always recommended to verify values before trading.
RoAnalytics was built to provide reliable, data-driven insights into item information - including predicted overpays for primarily high-demand items.

---

## Table of Contents
1. [Understanding Overpays in Trading](#1-understanding-overpays-in-trading)
2. [RAP vs Valued Items](#2-rap-vs-valued-items-in-overpay-estimates)
3. [Why Overpay is N/A](#4-why-is-the-overpay-estimate-na-not-available)
4. [Contact](#5-contact)

---

## 1. Understanding Overpays in Trading
When trading, there are two types of overpays to keep in mind: 
- **The overpay you give** - for example offering 300 overpay on a Legit
- **The overpay you recieve** - for example, recieving 600 overpay on a Legit

In an *ideal scenario*, you want to **give less overpay than you receive**. 
Because of this, overpay values can’t be represented by a single number/range; doing so would make it unclear whether that range applies to what you should give or what you should expect to receive.

---

## 2. RAP vs Valued Items in Overpay Estimates
- RAP Items
  - *Most* RAP items are valued purely by their Recent Average Price (RAP), meaning **their worth is directly tied to how frequently and how well they sell** on the market.
- Valued Items
  - Valued items are valued based on completed trades, offers, and demand trends (The rap can still affect their valuation, but not as much for items > 100K).

Because these two item types are valued differently, **their overpay estimates are also calculated differently**:
- RAP Items: rely on **price behavior and sale trends**.
  - RoAnalytics estimates overpays for these based on:
    - The current item’s RAP
    - The item's best price
    - How often the item sells (the demand)
  
- Valued Items: depend on **trading patterns, proof data, and market activity**.
  - RoAnalytics estimates overpays for these based on:
    - The item’s rap to value ratio
    - A prediction model, that is generated from a given set of data.

---

## 3. TO BE ADDED


---

## 4. Why is the Overpay Estimate N/A (Not Available)?
There are several reasons why RoAnalytics may not provide an overpay estimate for a specific item:
- Item value exceeds 160K
  - As the developer, I currently focus on items valued up to 160K. Items above this range aren’t yet supported because their trade data is more complex. As I gain more experience and collect more data, this limit will gradually increase.
- Items with low or terrible demand often have unpredictable overpays, caused by market manipulation or irregular trade activity.
  - For example, BPTH is known to have unstable overpay patterns, valued at 18K with 40K+ proofs.
  - The same logic applies to RAP-based items - if an item sells less than 0.1 times per day, its market activity is too low to form a reliable prediction. (Example: Aer, 45k RAP, proofs show 20-40K)
- Item is projected
  - If an item is flagged as projected on Rolimon’s, no overpay estimate will be calculated.
  - This prevents distorted or inflated data from being factored into predictions.

Code logic used for determining eligibility:
```python
    if (
      item_value > 160000 
      or demand in ["Low", "Terrible"] 
      or daily_sales <= 0.1
      or projected
    ):
```

---

## 5. Contact
If you have questions, please contact me!  
- Discord: **luxuryrangerover**

---

Thank you for using **RoAnalytics**!  
This project is still evolving - your feedback is VERY important to improve prediction accuracy and make trading insights more reliable.
