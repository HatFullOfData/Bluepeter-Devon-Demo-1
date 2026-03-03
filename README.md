# Power Query Transformations – Training Data

This repository contains sample data files for a **Power Query transformations** training session.  
The scenario is a Devon-based herbs & spices wholesaler supplying restaurants and caterers.

---

## Files

| File / Folder | Description |
|---|---|
| `customers.csv` | 20 restaurant and catering customers |
| `products.csv` | 30 herb and spice products |
| `orders/orders_2025_MM.csv` | 12 monthly order files for 2025 (one per month) |

---

## Transformation Exercises

The data has been deliberately prepared to require the following common Power Query transformations.

---

### 1. Trim Whitespace — `customers.csv` · `products.csv`

**What:** Some `CustomerName`, `ContactName`, and `Category` values contain leading and/or trailing spaces (e.g., `"  Moor Feast Caterers"`, `"Okehampton Catering  "`, `"Herb "`).

**Transform:** `Transform → Format → Trim` on the affected column(s).

**Result:** Clean text values with no unwanted spaces.

---

### 2. Split Column by Delimiter – Contact Name — `customers.csv`

**What:** The `ContactName` column stores first and last name together in one field (e.g., `"Sarah Mitchell"`), separated by a space.  
*(Note: some names also have an extra internal space, which makes Trim a useful prerequisite.)*

**Transform:** Select `ContactName` → `Home → Split Column → By Delimiter` → Space → Left-most delimiter.

**Result:** Two new columns: `ContactName.1` (first name) and `ContactName.2` (last name). Rename appropriately.

---

### 3. Split Column by Delimiter – Address — `customers.csv`

**What:** The `FullAddress` column stores the entire address as a single comma-delimited string  
(e.g., `"14 Quay Street, Dartmouth, Devon, TQ6 9PS"`).

**Transform:** Select `FullAddress` → `Home → Split Column → By Delimiter` → Comma → Each occurrence.

**Result:** Four new columns: street, town, county, postcode. Rename and trim each one.

---

### 4. Replace Values – Standardise CustomerType — `customers.csv`

**What:** The `CustomerType` column contains the same values in different cases:  
`"Restaurant"`, `"restaurant"`, `"RESTAURANT"`, `"Caterer"`, `"caterer"`, `"CATERER"`.

**Transform:** `Transform → Replace Values` (repeat for each variant), or use  
`Transform → Format → Capitalize Each Word` to normalise all values in one step.

**Result:** Consistent values of `"Restaurant"` and `"Caterer"` throughout.

---

### 5. Remove Currency Symbol & Change Type – Credit Limit — `customers.csv`

**What:** The `CreditLimit` column stores values as text with a pound sign and comma  
(e.g., `"£3,500"`, `"£12,000"`). Power Query cannot sum or average these until they are numeric.

**Transform:**  
1. `Transform → Replace Values` → replace `£` with nothing.  
2. `Transform → Replace Values` → replace `,` with nothing.  
3. `Transform → Data Type` → Whole Number.

**Result:** A numeric `CreditLimit` column ready for aggregation.

---

### 6. Remove Time from DateTime – Member Since — `customers.csv`

**What:** The `MemberSince` column stores dates with a time component  
(e.g., `"12/03/2018 09:00:00"`). For reporting you only need the date portion.

**Transform:** `Transform → Data Type` → Date (Power Query strips the time automatically).  
Alternatively: `Add Column → Date → Date Only`.

**Result:** A clean `Date` type column with no time component.

---

### 7. Split Column by Delimiter – Supplier Code — `products.csv`

**What:** The `SupplierCode` column contains a structured code such as `"SUPS-HRB-001"`,  
where the second segment indicates the category (`HRB` = Herb, `SPC` = Spice) and the  
third segment is a sequence number.

**Transform:** Select `SupplierCode` → `Home → Split Column → By Delimiter` → Hyphen (`-`) → Each occurrence.

**Result:** Three new columns: supplier prefix, category code, and sequence number.

---

### 8. Split Column by Position / Delimiter – Pack Size — `products.csv`

**What:** The `PackSize` column mixes numeric quantity and unit of measure in one field  
(e.g., `"50g"`, `"25 g"`, `"1 kg"`). Some have a space between number and unit; some do not.

**Transform (option A – digit boundary):**  
`Home → Split Column → By Non-Digit to Digit Transition` (or use a Custom Column with `Text.Select`).

**Transform (option B – space):**  
First use `Transform → Replace Values` to ensure a consistent space between number and unit  
(e.g., replace `"g"` with `" g"` and `"kg"` with `" kg"`), then split by space.

**Result:** Two columns: `PackQuantity` (number) and `PackUnit` (text).

---

### 9. Remove Currency Symbol & Trim – Unit Price — `products.csv` · `orders/`

**What:** `UnitPrice` is stored as text with a pound sign and sometimes a space after it  
(e.g., `"£ 2.80"`, `"£18.50"`).

**Transform:**  
1. `Transform → Replace Values` → replace `£` with nothing.  
2. `Transform → Format → Trim` to remove any leading/trailing spaces.  
3. `Transform → Data Type` → Decimal Number.

**Result:** A numeric price column.

---

### 10. Remove Percentage Symbol & Change Type – Discount — `orders/`

**What:** The `Discount` column contains values like `"10%"`, `"5%"`, `"0%"`, or blank.

**Transform:**  
1. `Transform → Replace Values` → replace `%` with nothing.  
2. `Transform → Replace Values` → replace empty string with `0` (to handle blank cells).  
3. `Transform → Data Type` → Decimal Number.  
4. `Add Column → Custom Column` → divide by 100 to convert to a true decimal proportion.

**Result:** A numeric discount column (e.g., 0.10 for 10%).

---

### 11. Remove Time from DateTime – Order Date — `orders/`

**What:** The `OrderDate` column includes a time component  
(e.g., `"01/01/2025 12:15:00"`).

**Transform:** `Transform → Data Type` → Date.

**Result:** A clean date column, ready for date-based grouping and filtering.

---

### 12. Change Type – Quantity (Text to Number) — `orders/`

**What:** The `Quantity` column is stored as text.

**Transform:** `Transform → Data Type` → Whole Number.

**Result:** A numeric column that can be used in calculations.

---

### 13. Replace Values – Standardise Status — `orders/`

**What:** The `Status` column contains the same values in multiple cases:  
`"Delivered"`, `"DELIVERED"`, `"delivered"`, `"Processing"`, `"PROCESSING"`,  
`"Despatched"`, `"DESPATCHED"`, `"Cancelled"`.

**Transform:** `Transform → Format → Capitalize Each Word` (or individual `Replace Values` steps).

**Result:** Consistent, title-case status values.

---

### 14. Remove Duplicate Rows — `orders/`

**What:** Each monthly orders file contains one intentionally duplicated row.

**Transform:** `Home → Remove Rows → Remove Duplicates`.

**Result:** Each order appears exactly once.

---

### 15. Add Custom Column – Line Total — `orders/`

**What:** There is no pre-calculated line total. After completing transformations 9, 10, and 12 above,  
delegates can calculate the net line value.

**Transform:** `Add Column → Custom Column`

```
= [Quantity] * [UnitPrice] * (1 - [Discount])
```

**Result:** A `LineTotal` column showing the net value of each order line.

---

### 16. Combine Files from Folder — `orders/`

**What:** There are 12 separate monthly CSV files in the `orders/` folder.  
A key Power Query skill is combining them into one table automatically.

**Transform:**  
1. In Power Query: `New Source → Folder` → select the `orders/` folder.  
2. Click `Combine & Transform`.  
3. Power Query applies the transformation steps to every file and appends all rows.

**Result:** A single `Orders` table containing all 2025 data (~450 rows).

---

## Quick Reference – Transformation Summary Table

| # | Column | File | Challenge | Transform |
|---|---|---|---|---|
| 1 | `CustomerName`, `ContactName`, `Category` | customers, products | Leading/trailing spaces | Trim |
| 2 | `ContactName` | customers | First + last name in one column | Split Column → Space |
| 3 | `FullAddress` | customers | Full address in one column | Split Column → Comma |
| 4 | `CustomerType` | customers | Inconsistent capitalisation | Replace Values / Capitalise |
| 5 | `CreditLimit` | customers | Currency symbol + comma in text | Replace Values → Change Type |
| 6 | `MemberSince` | customers | Date stored with time | Change Type → Date |
| 7 | `SupplierCode` | products | Structured code needs splitting | Split Column → Hyphen |
| 8 | `PackSize` | products | Number + unit in one field | Split Column → Non-digit |
| 9 | `UnitPrice` | products, orders | Currency symbol + space in text | Replace Values + Trim → Change Type |
| 10 | `Discount` | orders | Percentage symbol in text | Replace Values → Change Type → ÷ 100 |
| 11 | `OrderDate` | orders | Date stored with time | Change Type → Date |
| 12 | `Quantity` | orders | Number stored as text | Change Type → Whole Number |
| 13 | `Status` | orders | Inconsistent capitalisation | Replace Values / Capitalise |
| 14 | All columns | orders | Duplicate rows | Remove Duplicates |
| 15 | *(new)* `LineTotal` | orders | Calculated field | Add Custom Column |
| 16 | *(all)* | orders folder | Data split across 12 files | Combine Files from Folder |