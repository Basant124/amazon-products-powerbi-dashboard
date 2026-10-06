# 🛒 Amazon Products Analytics Dashboard (Power BI)

An interactive, five-page Power BI dashboard that analyses **1,402 Amazon products** across pricing, discounts, categories and customer ratings. The report is built around a custom dark-and-orange interface, including a home page written in HTML/CSS and rendered from a DAX measure.

---

## 📌 Project Goals

- Understand how products are priced and how deep discounts go.
- Compare main and sub-categories by volume, price, discount and rating.
- Find out whether bigger discounts or higher popularity come with better ratings.
- Deliver the analysis in a polished, app-like report with page navigation.

---

## 📂 Dataset

| Item | Detail |
|---|---|
| Source | Amazon Sales Dataset (Kaggle) |
| Products | 1,402 (after cleaning) |
| Main categories | 9 |
| Sub categories | 28 |
| Key fields | product name, category, actual price, discounted price, discount %, rating, rating count |
| Currency | INR (₹) |

### Cleaning steps (Power Query)
- Split the multi-level `category` path into **Main Category** and **Sub Category**.
- Removed currency symbols and thousands separators from price and `rating_count` columns, then converted them to numbers.
- Converted `discount_percentage` to a decimal percentage.
- Removed duplicate products and rows with missing ratings.
- Added helper columns: **Discount Bucket**, **Rating Group** and their sort-order columns.

---

## 📊 Report Pages

### 0. Home
A custom navigation page built with **HTML + CSS inside a DAX measure**, displayed through the HTML Content visual. It has an animated logo, a background image and a 4-tab carousel with working arrows, all without JavaScript.

### 1. Overview
![Overview](screenshots/01_overview.png)
- KPIs: Total Products, Avg Actual Price, Avg Discounted Price, Avg Discount %, Avg Rating
- Products by main category
- Avg Discount % vs Avg Rating by category
- Actual vs discounted price by category
- Category summary table

### 2. Pricing & Discount
![Pricing & Discount](screenshots/02_pricing_discount.png)
- KPIs: Max Actual Price, Max Discount %, Avg Discount Amount, % Products with Discount
- Avg discount amount by category
- Price vs Discount % scatter by category
- Top 10 most discounted products
- Products by discount level (donut)

### 3. Category Performance
![Category Performance](screenshots/03_category_performance.png)
- KPIs: Total Main Categories, Total Sub Categories, Top Category, Best Rated Sub Category
- **Rating vs Discount** scatter with average reference lines (quadrant view)
- **Decomposition tree** for main → sub category drill-down
- **Pareto chart** of the top 10 sub categories with cumulative %
- Category breakdown matrix with data bars and rating icons

### 4. Customer & Reviews
![Customer & Reviews](screenshots/04_customer_reviews.png)
- KPIs: Avg Rating, Total Ratings, Avg Ratings per Product, % Products Rated 4+
- Rating distribution histogram
- Rating mix by category (100% stacked bar)
- Top 10 most reviewed products
- Popularity vs Rating (columns = total ratings, line = avg rating)

---

## 💡 Key Insights

1. **Discounting is almost universal.** 96.5% of products are discounted, the average discount is 46.9%, and 46% of products are discounted by 50% or more. The deepest discount is 94%.
2. **Three categories dominate the catalogue.** Electronics, Computers & Accessories and Home & Kitchen hold about 97% of all products.
3. **Electronics is the premium category.** It has the highest average actual price (~₹10.3K) and the largest average discount amount (~₹3.9K per product).
4. **Ratings are high and tightly clustered.** The average rating is 4.10, 75.6% of products are rated 4 or above, and most ratings fall between 3.8 and 4.5.
5. **Bigger discounts tend to come with lower ratings.** Sub categories with discounts around 60% or more, such as Headphones and Wearable Technology, sit below the average rating. Low-discount sub categories, such as Craft Materials and Office Paper Products, rate above it.
6. **Popular does not mean best rated.** Headphones & Earbuds is the second most reviewed sub category (~4.7M ratings) but has the lowest rating in the top 10 (3.92).
7. **Customer engagement is concentrated.** The catalogue has ~26M ratings in total, averaging ~18.3K ratings per product.

---

## 🛠️ Technical Highlights

| Feature | How it was built |
|---|---|
| HTML home page | A DAX measure returns the full HTML/CSS string, shown with the **HTML Content** visual |
| Embedded images | Power Query reads image files from a folder, converts them to **Base64** and splits them into 30K-character chunks; DAX rebuilds them with `CONCATENATEX` |
| Carousel without JavaScript | Hidden radio inputs plus the CSS `:checked` selector move the tab track and highlight the active tab |
| Consistent design | Custom **theme JSON** and a designed **background image** for each page (1920×1080) |
| Navigation | Transparent buttons with page-navigation actions over the drawn tabs |
| Pareto chart | Cumulative % measure with a tie-breaker so equal values do not flatten the line |
| Fair rating comparisons | Ratings are only shown for sub categories with 5 or more products |
| Synced filtering | Main Category slicer synced across all pages |

### Sample DAX

```dax
Cumulative % Products =
VAR _CurVal  = [Total Products]
VAR _CurName = SELECTEDVALUE ( Fact_Products[Sub Category] )
VAR _Tbl =
    ADDCOLUMNS ( ALLSELECTED ( Fact_Products[Sub Category] ), "@P", [Total Products] )
VAR _Cum =
    SUMX (
        FILTER (
            _Tbl,
            [@P] > _CurVal
                || ( [@P] = _CurVal && Fact_Products[Sub Category] <= _CurName )
        ),
        [@P]
    )
RETURN
    DIVIDE ( _Cum, SUMX ( _Tbl, [@P] ) )
```

```dax
Best Rated Sub Category =
VAR _Eligible =
    FILTER ( VALUES ( Fact_Products[Sub Category] ), [Total Products] >= 5 )
VAR _Top =
    TOPN ( 1, _Eligible, [Avg Rating], DESC )
RETURN
    MAXX ( _Top, Fact_Products[Sub Category] )
```

---

## 📁 Repository Structure

```
├── Amazon_Dashboard.pbix
├── theme/
│   └── Amazon_Dark_Theme.json
├── backgrounds/
│   ├── Overview_Background.png
│   ├── Pricing_Discount_Background.png
│   ├── Category_Performance_Background.png
│   └── Customer_Reviews_Background.png
├── html/
│   └── index.html
├── data/
│   └── amazon.csv
├── screenshots/
│   ├── 00_home.png
│   ├── 01_overview.png
│   ├── 02_pricing_discount.png
│   ├── 03_category_performance.png
│   └── 04_customer_reviews.png
└── README.md
```

---

## ▶️ How to Use

1. Download or clone the repository.
2. Open `Amazon_Dashboard.pbix` in **Power BI Desktop** (latest version recommended).
3. If prompted, update the data source paths in **Power Query** to point to the `data/` folder and the images folder.
4. Click **Refresh**.
5. Use the tabs at the top of each page, or the home page carousel, to move between pages.

> The HTML home page needs the **HTML Content** custom visual from AppSource.

---

## 🧰 Tools

Power BI Desktop · Power Query (M) · DAX · HTML/CSS · Python (Pillow, for background and GIF assets)

---

## 👩‍💻 Author

**Basant** – Data Analyst
