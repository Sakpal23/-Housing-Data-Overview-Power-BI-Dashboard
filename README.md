# 🏠 Housing Data Overview | Power BI Dashboard

A 2-page Power BI report that shows how the housing market is doing: prices, sales, regions and sale types. Built with Power Query, DAX and Row-Level Security (RLS).

![Report Page 1](images/report-page-1.png)
![Report Page 2](images/report-page-2.png)

---

## 📌 Project Goal

Give a quick, clear view of the housing market and answer simple questions like:

- Which regions sell the most, and which are the most expensive per square metre?
- Are prices going up or down compared to last year?
- How far is the final purchase price from the offer price?
- Which sale type (regular, auction, family, other) is growing or falling?
- What affects the purchase price, for example the age of the house?

---

## 📊 Report Pages

### Page 1: House Market Overview
- Median Sales Price Change by Region
- Units Sold in the Latest Year and Quarter (card)
- Last 12 Month Sale (card)
- Offer Price vs Purchase Price (scatter chart)
- YOY Sales Growth by Sales Type

### Page 2: Sales Performance
- Average Price per SQM by Region (donut chart)
- Key Influencers (what drives the purchase price)
- Daily table with Year, Quarter, Month, Day, Total YTD Sales and Purchase Price
- Sales by Region (bar chart)
- Offer to Square Ratio by Sales Type

---

## 🔍 Key Findings

- **Zealand and Jutland lead the market.** Zealand has about 95bn in sales and Jutland about 81bn. Fyn & islands is about 15bn and Bornholm about 1bn.
- **Zealand is the most expensive per SQM** (around 20.85K). Bornholm is the cheapest (around 10.6K).
- **Median prices are down in every region** compared to the previous year, by roughly 25% to 32%.
- **Auction is the only sale type with positive YOY growth.** Family sale shows the biggest fall.
- **Purchase price follows offer price closely.** Most points sit on a straight line. A few outliers paid below the offer.
- **Regular sale has the highest offer-to-SQM ratio** (14.9K). Auction has the lowest (11.1K).
- **House age matters.** Key Influencers shows that very new houses (age 1 to 16) and very old houses (age over 69) have a higher average purchase price.
- The last 12 months of sales add up to about **13bn**.

---

## 🛠️ Tools and Features Used

| Area | What was done |
|---|---|
| Power Query | Fixed data types, added a conditional column, merged queries |
| Data Model | One main table (`Housing Data`) plus a separate `Measure Table-1` for all measures |
| DAX | 8 measures (listed below) and 2 calculated columns (`Age`, `Offer_Price`) |
| Security | Row-Level Security (RLS) so each user sees only the data they are allowed to see |
| Visuals | Donut, bar, scatter, area, table, cards, Key Influencers |
| Design | Dark green theme, 1920 x 1080 canvas |

---

## 🗂️ Data Fields

`house_id`, `date`, `quarter`, `region`, `city`, `area`, `address`, `house_type`, `sales_type`, `no_rooms`, `sqm`, `sqm_price`, `purchase_price`, `Offer_Price`, `%_change_between_offer_and_purchase`, `year_build`, `Age`, `dk_ann_infl_rate%`, `nom_interest_rate%`, `yield_on_mortgage_credit_bonds%`

![Data Model](images/data-model.png)

---

## 🧮 DAX Measures

**1. Average Price Per Meter**
```DAX
Average_Price_Per_Meter = AVERAGE('Housing Data'[sqm_price])
```

**2. Last 12 Month Sale**
```DAX
Last_12_Month_Sale =
CALCULATE(
    SUM('Housing Data'[purchase_price]),
    DATESINPERIOD('Housing Data'[date], MAX('Housing Data'[date]), -12, MONTH)
)
```

**3. Median Sales Price Change** (latest year vs previous year)
```DAX
Median_Sales_Price_Change =
VAR Currn_Median_Price =
    MEDIANX(
        FILTER(
            'Housing Data',
            YEAR('Housing Data'[date].[Date]) = YEAR(MAX('Housing Data'[date].[Date]))
        ),
        'Housing Data'[purchase_price]
    )
VAR Previouse_Year_Median_Price =
    MEDIANX(
        FILTER(
            'Housing Data',
            YEAR('Housing Data'[date].[Date]) = YEAR(MAX('Housing Data'[date].[Date])) - 1
        ),
        'Housing Data'[purchase_price]
    )
RETURN
    IF(
        Previouse_Year_Median_Price <> 0,
        (Currn_Median_Price - Previouse_Year_Median_Price) / Previouse_Year_Median_Price,
        BLANK()
    )
```

**4. Offer To SQM Ratio**
```DAX
Offer_To_SQM_Ration = DIVIDE(SUM('Housing Data'[Offer_Price]), SUM('Housing Data'[sqm]))
```

**5. Sales By Region**
```DAX
Sales_By_Region =
CALCULATE(
    SUM('Housing Data'[purchase_price]),
    ALLEXCEPT('Housing Data', 'Housing Data'[region])
)
```

**6. Total YTD Sales**
```DAX
Total_YTD_Sales = TOTALYTD(SUM('Housing Data'[purchase_price]), 'Housing Data'[date].[Date])
```

**7. Units Sold (Latest Year and Quarter)**
```DAX
Units_Sold_Latest_Year_&_Quarter =
CALCULATE(
    DISTINCTCOUNT('Housing Data'[house_id]),
    YEAR('Housing Data'[date]) = YEAR(MAX('Housing Data'[date])) &&
    QUARTER('Housing Data'[date]) = QUARTER(MAX('Housing Data'[date]))
)
```

**8. YOY Sales Growth**
```DAX
YOY_Sales_Growth =
VAR Current_Sales =
    CALCULATE(
        SUM('Housing Data'[purchase_price]),
        YEAR('Housing Data'[date]) = YEAR(MAX('Housing Data'[date]))
    )
VAR Previous_Sales =
    CALCULATE(
        SUM('Housing Data'[purchase_price]),
        YEAR('Housing Data'[date]) = YEAR(MAX('Housing Data'[date])) - 1
    )
RETURN
    IF(
        Previous_Sales <> 0,
        (Current_Sales - Previous_Sales) / Previous_Sales,
        BLANK()
    )
```

---

## 🔐 Row-Level Security (RLS)

RLS is set up in Power BI Desktop (**Modeling > Manage roles**). Each role has a DAX filter on the `Housing Data` table, so a user only sees the rows they are allowed to see. Use **View as** to test a role before publishing.

---

## 📁 Repository Structure

```
Housing-Data-Overview/
│
├── Housing_Data_Overview.pbix
├── Housing_Data_Overview.pptx
├── README.md
└── images/
    ├── report-page-1.png
    ├── report-page-2.png
    └── data-model.png
```

---

## ▶️ How to Open

1. Download `Housing_Data_Overview.pbix`.
2. Open it in **Power BI Desktop** (free).
3. Use **View as** on the Modeling tab to test the RLS roles.

---

## 🚀 Possible Improvements

- Add a proper Date table and use it for all time measures.
- Compare the same period of both years, so a part-year does not look like a fall.
- Add a region and sale type slicer to Page 1.

---

## 👤 Author

**Umesh Sakpal**
Power BI Developer and Data Analyst

📧 umeshsakpal14@gmail.com
🔗 [GitHub](https://github.com/Sakpal23)

⭐ If you found this useful, give the repo a star.
