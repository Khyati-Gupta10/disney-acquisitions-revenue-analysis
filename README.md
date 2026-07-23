<img width="681" height="383" alt="image"  src="https://github.com/user-attachments/assets/5f2982fc-bf08-4c60-b823-28c08e02e5e5"  / >

# Disney Acquisitions & Revenue Analysis

**A SQL-Based Exploration of Disney's Major Acquisitions and Reported Revenue (1957–2024)**

> **Scope note:** Acquisition history covers 1957–2019; revenue history covers 2009–2024 only (the years with directly comparable consolidated revenue figures). Figures are drawn from public sources; this is an educational/portfolio project, not a peer-reviewed financial study.

---

## Abstract

This project builds a small relational SQL database of Disney's major acquisitions and annual consolidated revenue, then uses validation checks and analytical queries (window functions, CTEs) to explore year over year revenue growth alongside the timing of landmark deals (e.g., Pixar, Marvel, Capital Cities/ABC, 21st Century Fox). It is intended as a demonstration of SQL technique ,data modeling, validation, and window-function analysis rather than a causal study of M&A performance.

---

## 1. Introduction

Mergers and acquisitions (M&A) are often cited as pivotal moments in corporate strategy. Disney's history  from its early Disneyland ownership consolidation in the late 1950s to the $71.3 billion acquisition of 21st Century Fox in 2019 offers a well-known case study. This project uses SQL to:

1. Catalogue Disney's major acquisitions.
2. Track annual revenue over the 2009–2024 window for which comparable figures exist.
3. Demonstrate SQL techniques (CTEs, window functions, validation queries) applied to a small financial dataset.

**Important limitation:** revenue changes shown here are correlated with acquisition timing, not proven to be *caused* by it. In particular, 2020–2022 revenue movement overlaps heavily with COVID-19 park/theatrical closures and reopening, which is a major confound with any acquisition-driven synergy story (see Discussion).

---

## 2. Literature Review

- **Corporate M&A Performance**: Large-scale acquisitions can yield both synergies and integration risks (Capron & Mitchell, 2009).
- **Disney's Strategic Growth**: Disney's IP-driven acquisitions Pixar (2006), Marvel (2009), Lucasfilm (2012) are widely discussed as a content-strategy pattern (Johnson, 2018).
- **SQL in Business Research**: SQL is a practical tool for managing and querying longitudinal financial data (Kim et al., 2020).

---

## 3. Data & Schema

### 3.1 Data Sources

- **Acquisitions**: Acquisition name, date, country, original & inflation adjusted price, parent division, source reference.
- **Revenue**: Annual consolidated revenue, 2009–2024, in millions USD.

### 3.2 Database Design

```sql
CREATE TABLE acquisitions (
  id INT PRIMARY KEY AUTO_INCREMENT,
  company_acquired VARCHAR(255),
  date DATE,
  country VARCHAR(255),
  price DECIMAL(15,2),          -- raw USD
  price_adjusted DECIMAL(15,2), -- raw USD, inflation-adjusted
  parent_merged_with VARCHAR(255),
  reference VARCHAR(255)
);

CREATE TABLE revenue (
  year INT PRIMARY KEY,
  revenue DECIMAL(15,2) -- millions USD
);
```

**Note on units:** `acquisitions.price`/`price_adjusted` are stored in raw USD, while `revenue.revenue` is stored in millions USD. The two are never directly comparable without converting units first.

---

## 4. Methodology

### 4.1 Schema Deployment

Executed the DDL in `disney-analysis.sql` to drop and recreate the schema:

- **Database:** `disney_analysis`
- **Tables:** `acquisitions`, `revenue`

### 4.2 Data Insertion

- **Acquisitions:** 7 deals spanning 1957–2019.
- **Revenue:** Annual figures for 2009–2024.

These figures reflect publicly reported historical data used for this exercise, not synthetic/placeholder values.

### 4.3 Validation

1. **Duplicate Check:** Queried `revenue` for repeated `year` entries.
2. **Null Check:** Checked `acquisitions` for missing `date`/`price`/`price_adjusted`.

### 4.4 Analytical Queries

Applied CTEs and window functions (`LAG()`, `SUM() OVER`) to compute year-over-year growth and cumulative acquisition/revenue trends.

---

## 5. Results

### 5.1 Acquisition Timeline

| Year | Company                  | Adjusted Price (USD) |
| ---- | ------------------------ | --------------------- |
| 1957 | Disneyland (64% stake)   | 6.00 million           |
| 1960 | Disneyland (remaining)   | 80.00 million          |
| 1996 | Capital Cities/ABC       | 38.09 billion          |
| 2006 | Pixar                    | 11.54 billion          |
| 2009 | Marvel Entertainment     | 6.45 billion           |
| 2012 | Lucasfilm                | 5.55 billion           |
| 2019 | 21st Century Fox         | 87.69 billion          |

### 5.2 Revenue Growth (2009–2024)

| Year | Revenue (M USD) | YoY Growth (%) |
| ---- | ---------------- | ---------------- |
| 2009 | 36,149           | —                |
| 2010 | 38,063           | +5.29            |
| 2011 | 40,893           | +7.44            |
| 2012 | 42,278           | +3.39            |
| 2013 | 45,041           | +6.54            |
| 2014 | 48,813           | +8.37            |
| 2015 | 52,465           | +7.48            |
| 2016 | 55,632           | +6.04            |
| 2017 | 55,137           | -0.89            |
| 2018 | 59,434           | +7.79            |
| 2019 | 69,607           | +17.12           |
| 2020 | 65,388           | **-6.06**        |
| 2021 | 67,418           | +3.10            |
| 2022 | 82,722           | +22.70           |
| 2023 | 88,898           | +7.47            |
| 2024 | 91,361           | +2.77            |

2020's decline (pandemic driven park and theatrical closures) is the largest single year drop in the dataset, followed by the largest single year rebound in 2022. Both are far larger in magnitude than the more gradual growth seen around the Pixar/Marvel/Lucasfilm deals.

---

## 6. Discussion

**2020–2022 is dominated by COVID-19, not M&A synergy.**
The steep 2020 decline and the 2022 rebound line up closely with pandemic closures and reopening of parks, cruises, and theatrical releases not with the timing of the Fox acquisition itself (completed March 2019, a full year before the decline). This dataset cannot separate a "Fox integration" effect from a "pandemic recovery" effect, since both occurred in the same window. A fair reading is: the data is *consistent with* continued post Fox integration, but the pandemic is the more parsimonious explanation for the shape of the 2020–2022 swing specifically.

**Content led growth (Pixar, Marvel, Lucasfilm) shows a steadier pattern.**
The 2010–2019 period shows consistent mid-to-high single-digit growth, with no single year standing out sharply around the Pixar (2006), Marvel (2009), or Lucasfilm (2012) deals individually consistent with these being smaller, more gradually integrated acquisitions relative to Fox.

**Methodological limitations:**
- No control group (e.g., industry-wide media revenue) to separate acquisition effects from broader market trends.
- Revenue data only starts in 2009, so acquisitions before then (Disneyland, Capital Cities/ABC, Pixar) cannot be compared against a "before" revenue baseline in this dataset.
- Correlation in timing is not evidence of causation; no statistical test (e.g., difference-in-differences) has been applied here.

---

## 7. Conclusion

This project demonstrates SQL techniques schema design, validation, window functions, and CTEs applied to a small acquisitions-and-revenue dataset. It surfaces a clear, well-documented revenue trend from 2009–2024, including the 2020 pandemic-driven dip and 2022 rebound, but stops short of establishing that any single acquisition *caused* a given change in revenue. That would require the extensions noted below.

---

## 8. Future Work

1. **Extend revenue history back to 1957** so all acquisitions can be evaluated against a "before" baseline, not just Marvel/Lucasfilm/Fox.
2. **Add a control series** (e.g., broader media/entertainment industry revenue index) to separate acquisition effects from macro trends like COVID-19.
3. **Statistical significance testing** (e.g., difference-in-differences, regression discontinuity) rather than eyeballing growth rate tables.
4. **Interactive dashboards** in Tableau/Power BI for exploring acquisition timing against revenue interactively.

---

## 9. References

1. Capron, L., & Mitchell, W. (2009). *Build, Borrow, or Buy: Solving the Growth Dilemma*. Harvard Business Press.
2. Johnson, L. (2018). "Disney's Acquisition Strategy: From Pixar to Fox." *Journal of Media Economics*, 31(2), 59–74.
3. Kim, S., Smith, J., & Lee, H. (2020). "SQL for Financial Data Analysis." *International Journal of Data Science*, 5(1), 45–61.
