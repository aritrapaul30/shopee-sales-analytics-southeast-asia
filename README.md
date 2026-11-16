# 🛒 Shopee Sales Analytics: Electronics & Fashion Across Six Southeast Asian Markets

---

## 📌 Project Overview

Shopee runs on transaction records. Every order placed through the app leaves behind a row of product, price, customer and seller detail, but that data only becomes useful to pricing, inventory, promotions and seller-quality teams once it has been cleaned and read properly.

This project takes two raw Shopee transaction datasets, one for **consumer electronics** (Malaysia, Singapore, Indonesia) and one for **fashion & apparel** (Thailand, Philippines, Vietnam), and builds a **full data cleaning and exploratory analysis pipeline** in Python to answer three questions:
1. **What is wrong with the raw data, and how can it be fixed defensibly?**
2. **How do the two product lines and six markets behave from 2019 to 2023?**
3. **Which factors actually drive order value?**

---

## 🗃️ Dataset

| Property | Detail |
|---|---|
| **Source** | Shopee transaction datasets (Electronics & Fashion) |
| **Records** | 2,244 raw rows per dataset → **2,200 clean orders** each |
| **Collection Window** | 2019 - 2023 (pre-COVID, pandemic and recovery periods) |
| **Electronics Markets** | Malaysia · Singapore · Indonesia |
| **Fashion Markets** | Thailand · Philippines · Vietnam |
| **Columns** | Electronics: 20 · Fashion: 22 (adds `size` and `color`) |
| **Data Quality** | Raw files contain negatives, invalid discounts & ratings, mixed date formats, inconsistent region labels, duplicates and missing values |

### 📋 Feature Descriptions

| Variable | Type | Description |
|---|---|---|
| transaction_id | Categorical | Unique identifier for each order |
| seller_id | Categorical | Unique identifier for the seller |
| product_id | Categorical | Unique identifier for the product |
| product_name | Categorical | Name of the product sold |
| category | Categorical | Product category (e.g. Headphones, Shoes) |
| brand | Categorical | Product brand |
| region | Categorical | Market where the order was placed |
| order_date | Date | Order date (stored as text in the raw data) |
| size | Categorical | Apparel size - **Fashion only** |
| color | Categorical | Apparel colour - **Fashion only** |
| quantity | Numeric | Units bought in the order |
| unit_price | Numeric | Price of a single unit |
| total_price | Numeric | Total order value - **key sales measure** |
| discount_pct | Numeric | Discount applied, as a percentage |
| shipping_fee | Numeric | Delivery charge for the order |
| payment_method | Categorical | e.g. ShopeePay, Credit Card, Bank Transfer |
| order_status | Categorical | e.g. Completed, Cancelled |
| customer_rating | Numeric | Customer rating on a 1 - 5 scale |
| review_count | Numeric | Number of reviews recorded for the product |
| is_flash_sale | Boolean | Whether the order was part of a flash sale |
| shopee_coins_earned | Numeric | Loyalty coins earned on the order |
| seller_rating | Numeric | Seller rating on a 1 - 5 scale |

---

## 🔧 Data Cleaning & Preparation

### Issues Found in the Raw Data

| Issue | Electronics | Fashion |
|---|---|---|
| Negative `quantity` | 45 | 45 |
| Negative `total_price` | 44 | 44 |
| `discount_pct` outside 0 - 100 | 22 | 22 |
| `customer_rating` above 5 | 22 | 22 |
| Dates in `dd/mm/yyyy` instead of `yyyy-mm-dd` | 89 | 89 |
| Region label variants (for 3 countries) | 14 | 12 |
| Fully duplicated rows | 44 | 44 |
| **Column with most issues** | **customer_rating (174)** | **customer_rating (182)** |

### Cleaning Pipeline

| Step | Treatment | Reason |
|---|---|---|
| Negative quantity & price | Converted to absolute values | Most negatives sit under a *Completed* status with positive unit price and shipping fee, so they read as sign-entry errors, not refunds |
| Invalid discounts | Recomputed from the pricing formula (only where valid) | The other price fields already fix what the discount must be |
| Order date | Parsed both formats explicitly into one `datetime` | Enables year and trend analysis; 0 failed conversions |
| Region labels | Mapped codes → country name, stripped, lower-cased | `Malaysia`, `MY`, `MYS`, `malaysia` all collapse to one value |
| Duplicates | Dropped full-row and repeated `transaction_id` rows | 2,244 → 2,200 rows; one row per order |
| Ratings above 5 | Set to missing, then imputed | The true score behind an impossible value is unknowable |
| Missing values | Rule-based imputation (below) | Each rule matches what the field means |

**Pricing formula used to repair and validate discounts:**

```
Discount % = 100 × (1 − (Total Price − Shipping Fee) ÷ (Quantity × Unit Price))
Total Price = Quantity × Unit Price × (1 − Discount % ÷ 100) + Shipping Fee
```

> After cleaning, **all 2,244 rows in each dataset matched the pricing formula**, which validates both the absolute-value correction and the recomputed discounts in one check.

### Missing-Value Strategy

| Case | Imputation |
|---|---|
| Both `customer_rating` and `review_count` missing | Product never reviewed → both set to **0** |
| Only `customer_rating` missing | Median rating of the same product, else overall median |
| Only `review_count` missing | Median review count of products with a similar rating |
| `seller_rating` missing | Median rating of the same seller |
| `shopee_coins_earned` missing | 0 |
| Other numeric / text / boolean | Column median / `Unknown` / `False` |

> Result: **zero negatives, zero missing values, valid ratings, one clean date type and one label per country** in both datasets, saved as `shopee_electronics_clean.csv` and `shopee_fashion_clean.csv`.

---

## 📊 Exploratory Data Analysis - Key Findings

### Two Very Different Business Models

| Statistic | Electronics | Fashion |
|---|---|---|
| Median quantity per order | 6 (max 10) | 8 (max 15) |
| Median unit price | ~557 | ~67 |
| Max unit price | ~3,000 | ~250 |
| Median order value | ~1,946 | ~325 |
| Average discount | ~20% (max 40%) | ~25% (max 50%) |
| Average shipping fee | ~10 | ~4 |

> Electronics is a **low-volume, high-value** business. Fashion is a **high-volume, low-value** business with deeper discounting. Electronics order value is right-skewed: the mean (~3,654) is nearly **2× the median**, pulled up by a small number of large orders.

### Top Products & Categories

| Measure | Electronics | Fashion |
|---|---|---|
| 🥇 Top product (units) | Samsung Galaxy Tab A8 - 442 | Barong Tagalog - 609 |
| 🥈 2nd | Xiaomi Redmi Buds 4 - 421 | Toddler Romper - 558 |
| 🥉 3rd | AirPods Pro 2 - 419 | Gym Hoodie - 545 |
| Most transactions | Headphones - 336 orders | Men's Clothing - 327 orders |
| Most units sold | Headphones - 1,914 | Shoes - 2,634 |
| Fewest units sold | Cameras - 1,566 | Sportswear - 2,381 |

> Category demand is **spread evenly** in both markets: first to last is under 350 units in Electronics and about 250 in Fashion. No single category carries the business.

### Orders by Year

| Year | Electronics | Fashion |
|---|---|---|
| 2019 | 423 | 440 |
| 2020 | 453 | 453 |
| 2021 | 436 | 431 |
| **2022** | **465** | **458** |
| 2023 | 423 | 418 |

> 2022 was the busiest year in both markets. The pandemic years **did not** produce a spike in order counts.

### Total Sales by Country (2019 - 2023)

| Electronics | Total Sales | Fashion | Total Sales |
|---|---|---|---|
| **Malaysia** | **~3.08M** | **Thailand** | **~391K** |
| Singapore | ~2.61M | Vietnam | ~341K |
| Indonesia | ~2.35M | Philippines | ~324K |

---

## 🎨 Plot Evaluation & Improvement

The Task 3 country bar chart is accurate but **collapses five years into one bar**, hiding the fact that Malaysia peaked at **733K in 2022** and then fell to **506K in 2023**, behind Indonesia (523K). Three alternatives were built with year on the x-axis:

| Plot | Strength | Weakness | Verdict |
|---|---|---|---|
| Grouped bar chart | Easy country comparison within a year | 15 bars, hard to follow one country | Fine for single-year comparisons |
| **Multi-line chart** | Trends and crossovers read instantly | Exact values less visible | ✅ **Best for "sales by country over time"** |
| Heatmap (country × year) | Pattern plus every exact value | Less intuitive for direction | Close second when exact figures matter |

---

## 📈 Sales Trend Analysis

### Best-Selling Brands (Electronics, by Units)

| Rank | Malaysia | Indonesia |
|---|---|---|
| 🥇 1 | **Logitech - 434** | **Xiaomi - 291** |
| 🥈 2 | Oppo - 348 | Samsung - 288 |
| 🥉 3 | Apple - 346 | Apple - 274 |

> Apple is the only brand in both top-3 lists. The leaders differ (accessories-led Logitech vs value-led Xiaomi), so **brand strategy should be set country by country**.

### Sales Consistency (Coefficient of Variation of Yearly Sales)

| Rank | Country | CV | Reading |
|---|---|---|---|
| 🥇 1 | **Thailand** | **0.05** | Most stable (~72K - 81K every year) |
| 2 | Vietnam | 0.07 | Stable |
| 3 | Philippines | 0.07 | Stable |
| 4 | Singapore | 0.09 | Moderate |
| 5 | Indonesia | 0.13 | Volatile |
| 6 | Malaysia | 0.15 | Most volatile (545K → 733K → 506K) |

> CV ignores scale, so a smaller Fashion market and a larger Electronics market can be compared fairly.

### Correlation Among Features

| Pair | Electronics | Fashion | Reading |
|---|---|---|---|
| unit_price ↔ total_price | **0.79** | **0.69** | Strongest driver of order value |
| shopee_coins_earned ↔ total_price | 0.61 | 0.58 | Coins scale with spend |
| quantity ↔ total_price | 0.44 | 0.54 | Second core driver |
| discount_pct ↔ total_price | -0.14 | -0.19 | Heavier discounts trim order value slightly |
| ratings, reviews, shipping, year | ≈ 0 | ≈ 0 | No meaningful linear relationship |

> Flash-sale and non-flash-sale orders show **near-identical order-value distributions**, and average customer rating sits between **2.96 and 3.01 in every market**.

### Business Recommendations

| Insight | Finding | Action |
|---|---|---|
| Clean before you count | Negatives, duplicates and impossible ratings would distort every metric | Validate totals against the pricing formula as a standard data-quality check |
| Two different businesses | Electronics = high value per order; Fashion = high volume, deeper discounts | Plan inventory, discounting and KPIs separately per product line |
| Stable vs volatile markets | Thailand CV 0.05 vs Malaysia CV 0.15 | Forecast against steady markets; monitor Malaysia closely |
| Totals hide trends | Malaysia leads overall but fell behind Indonesia in 2023 | Always review country sales over time, not only as totals |
| Brand preference is local | Logitech leads Malaysia, Xiaomi leads Indonesia | Set brand and promotion strategy per country |
| Price and volume drive value | Flash sales and ratings barely move order value | Focus growth levers on pricing, basket size and loyalty coins |

---

## 📁 Repository Structure

```
├── Dataset/
│   ├── shopee_electronics_data.csv   # Raw Electronics transactions
│   └── shopee_fashion_data.csv       # Raw Fashion transactions
├── Outputs/                          # Charts and tables produced by the notebook
├── Python Code/
│   └── Python_Notebook.ipynb         # Full cleaning + analysis pipeline
├── Report/
│   └── Project Report.pdf            # 35-page written report
├── README.md
└── requirements.txt
```

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas)
![NumPy](https://img.shields.io/badge/NumPy-1.x-013243?logo=numpy)
![Matplotlib](https://img.shields.io/badge/Matplotlib-3.x-11557c)
![Seaborn](https://img.shields.io/badge/Seaborn-0.13-blue)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![Google Colab](https://img.shields.io/badge/Google-Colab-F9AB00?logo=googlecolab&logoColor=white)

**Key techniques:** Data validation · Formula-based imputation · Regex date parsing · Group-wise median imputation · `drop_duplicates` · `groupby` aggregation · Coefficient of variation · Correlation analysis · Pandas `Styler` tables · Seaborn bar, line, box & heatmap plots

---

## 📬 Contact

**Aritra Paul** - Master of Business Analytics, Sunway University  
📧 aritrapaul30@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/aritrapaul30)
