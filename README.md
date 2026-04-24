# MIST4610-Group-Project-2--Group-5

## Team Name: 
47114 Group B5 

## Team Members:

1. Aneesh Rudaraju (Conceptual Modeler)
2. Rory Drew (SQL Writer)
3. Kevin Duong (Database Designer/Data Wrangler)
4. Ved Doddapaneni (Group Leader)

## Case Summary

Northline Outfitters is a small online retail company that sells student-friendly lifestyle and tech accessories — including hoodies, water bottles, desk lamps, phone cases, keyboards, mouse pads, and backpacks — directly to consumers in the United States and Canada. The company purchases merchandise from outside vendors and resells it through an online storefront. Because the business is still growing, most of its records have been maintained in Excel rather than a formal database system. We were provided with two spreadsheets exported from the company's operations: **Sales_Dump**, containing 200 rows of order line data, and **Product_Supplier_Master**, containing product and vendor reference data. The data presented several challenges including mixed date formats, numeric fields stored as strings, and inconsistent category labels — all of which are documented in detail in the Data Quality Assessment section below.

## Data Model
<img width="1253" height="1089" alt="IMG_3056" src="https://github.com/user-attachments/assets/803ce013-1ab8-4bf7-96c9-6193e5375601" />

### Entities and Relationships

The database contains 8 entities designed to capture Northline Outfitters' sales operations, product catalog, and supplier relationships.

**Employee** stores the staff members who process orders. Each employee reports to one **Manager**, and a manager can oversee many employees.

**Manager** represents a supervisory employee. A manager oversees one or more employees and is linked to the employees beneath them.

**Order** represents a customer transaction. Each order is placed by one **Customer**, processed by one **Employee**, and contains one or more order lines. It stores the sale date, payment method, shipping country, and shipping destination.

**OrderLine** is the central fact entity. Each line belongs to one **Order** and references one **Product**. It stores quantity, unit price, discount, tax, line total, size or weight, return flag, and any notes tied to that line.

**Customer** stores the buyer's name, email, and customer type (Standard, Student, Loyalty, or Guest), which was extracted from the raw `customer_info` field during cleaning.

**Product** represents a single SKU in the catalog. Each product belongs to one **Category** and is supplied by one **Vendor**. A product can also reference a parent product via `parent_sku` to represent size or color variants.

**Category** is a controlled lookup table of product categories: Tech, Apparel, Accessories, Lifestyle, School, Audio, and Desk Setup.

**Vendor** stores supplier information including vendor name, phone number, and sales rep contact. One vendor can supply many products.
# Data Quality Assessment & Data Cleaning Process

## Data Quality Assessment

### Sales_Dump Issues

**Date inconsistencies** — The sale_date column used at least 8 different formats across all 200 rows, it included the following formats all within the same column: U.S. numeric (10-11-2025), European numeric (31/10/2025), abbreviated (Oct 17 25), and fully written styles.

**SKU inconsistencies** — 18 of 200 rows stored SKUs in all-lowercase (e.g., `sku-c-1012`) while the rest used uppercase (`SKU-C-1012`). A case-sensitive JOIN on SKU would silently drop these 18 rows.

**Country inconsistencies** — The field contained `US`, `USA`, `CA`, and `Canada`. Three rows had no country value at all.

**Payment inconsistencies** — The same payment methods appeared in multiple casings: `visa`, `VISA`, `debit`, `Debit`, and `MC` alongside `Mastercard`.

**Quantity stored data type** — 31 rows stored quantity as the string `'2 units'` instead of the integer `2`, making the column a mixed type and preventing aggregation.

**Currency code embedded in unit_price** — All 200 rows stored unit price as a string like `'USD 18.99'` or `'CAD 46.99'` making the column non-numeric with mixed values.

**Discount format inconsistencies** — Discounts appeared as percent strings with label (`'10%'`), labeled student percent (`'student 10%'`), percent strings (`'5%'`), promo code text (`'promo5'`), bare integer (`'5'`), zero (`'0'`).

**Tax format inconsistencies plus 33 NULLs** — Tax values appeared in six distinct non-null forms: percent strings (`'7%'`, `'8.25%'`, `'13%'`), decimals (`0.07`, `0.0825`, `0.13`), and a label-prefixed string (`'HST 13%'`). There were an additional 33 rows with no tax value at all.

**Line total inconsistencies** — 72 rows had a `'$'` prefix making the field non-numeric, and 62 rows had no total value at all, meaning revenue cannot be directly summed from this column.

**Return flag inconsistencies** — 40 of 200 rows have no return flag. Defaulting these to `'N'` would be analytically incorrect since missing data is meaningfully different from a confirmed non-return.

**customer_info has too much info** — All 200 rows stored customer name, loyalty status, student status, and guest status in one field, using three different delimiters (`|`, `;`, `/`).

**Incorrect email address** — Row LN-041 contains `'emma_wilson@school'`, which is missing its top-level domain and would fail any email validation check.

**Category customer type conflated with product category** — 75 rows had values like `'Accessories / Student'` or `'Desk Setup / Student'`, conflating a product attribute with a customer attribute.

---

### Product_Supplier_Master Issues

**Duplicate SKU rows for product variants** — The same SKU appears on multiple rows representing color or size variants (e.g., SKU-U-1005 appears 5 times). There is no consistent parent-child structure, making it unclear which row is authoritative for price, weight, and category.

**Reorder level inconsistencies** — 4 rows contain the word `'ten'` instead of the integer `10`, making the column a mixed type and preventing numeric threshold comparisons.

**Vendor phone inconsistencies** — Phone numbers appear in four formats: standard dashes (`404-555-0181`), parentheses (`(404)555-0181`), dots between all segments (`512.555.0173`), and dots between area code and number (`604.555.0190`).

**Vendor rep inconsistencies** — Entries like `'Jason Wu / email missing'` include a note, `'Mia Dia zFernandez'` contains a typo, and `'Ms. Anika Roy'` uses an inconsistent honorific prefix.

**Cost and list price inconsistencies** — Some rows include a currency prefix (`'USD 6.25'`, `'CAD 89.99'`) while others use bare numbers (`'31.4'`, `'27.8'`), making both columns mixed-type strings.

**Weight inconsistencies** — Values span four unit types with inconsistent abbreviations: ounces (`'8 ounces'`, `'8 oz'`), pounds (`'1.1 pound'`, `'1.1 lb'`, `'1.1 lbs'`), kilograms (`'0.22 kilograms'`, `'0.22kg'`), and grams (`'340 grams'`, `'340 g'`, `'450g'`). No standard format was enforced.

**Length inconsistencies** — Length values switch between imperial (`'10.2 in'`, `'11.5"'`) and metric (`'25.4 cm'`, `'29.2 centimetres'`) with no consistent unit across rows.

**Category inconsistencies** — Entries include `'Lifestyle , Student'` (comma separator), `'Tech & Student'` (ampersand), `'Student and apparels'` (freeform), and `'Accessories / Accessories'` (redundant duplicate).

**Discontinued has 17 NULL values** — 17 of 60 product rows have no discontinued flag, creating ambiguity about whether those products are active or simply unrecorded.

---

### Overall Data Quality Assessment

The very clear main issue with the data is the lack of consistency with the inputs. The source data reflects the typical state of records kept where no formal data entry standards were enforced. Both files suffer from the same underlying problem: fields were used as free-text notes rather than structured data. Employees entered values in whatever format felt natural at the time, producing a dataset where nearly every column requires some form of parsing or standardization before it can be used in a database or analytical query.

The Sales_Dump sheet is considerably more problematic. The core numeric fields — `unit_price`, `discount`, `tax`, and `line_total` — are all stored as strings due to embedded currency symbols, percent signs, and text labels, making them nearly impossible to work with in SQL. The `customer_info` field is a first normal form violation that encodes multiple distinct attributes in one column. The date formats are split into so many different formats that they are impossible to work with as-is. Taken together, these issues mean that no meaningful aggregation could be performed on the raw data without preprocessing.

The Product_Supplier_Master has a different but equally serious problem: the same SKU is reused across multiple rows for product variants, with no enforced parent-child structure. This makes it impossible to answer basic questions like "what is the list price of SKU-C-1002?" without first deciding which row is the primary record. Measurement fields (weight and length) use a mix of imperial and metric units with no standard, and the category field was treated as a freeform text area rather than a controlled vocabulary.

In summary, while the data contains all the information needed to build a functional retail database, it cannot be used in its raw state. The inconsistencies affect every table and nearly every column, and reflect a lack of input validation at the point of data entry rather than isolated mistakes.

---

## Data Cleaning Process

### Sales_Dump Fixes

**1. Date inconsistencies**

A multi-pass parser using Python's `pd.to_datetime()` was applied with a regex pre-processor that first expanded two-digit years (`'Oct 17 25'` → `'Oct 17 2025'`). The parser attempted both month-first and day-first interpretation as a fallback to handle the U.S./European ambiguity. All dates were normalized to ISO 8601 format (YYYY-MM-DD). The single 2026 date was retained but flagged in a new `data_flag` column with the value `FUTURE_DATE`.

| Before (Raw) | After (Cleaned) |
|---|---|
| 10-11-2025 (U.S. numeric) | 2025-10-11 |
| 31/10/2025 (European numeric) | 2025-10-31 |
| Oct 17 25 (abbreviated year) | 2025-10-17 |
| October 5 2025 (written out) | 2025-10-05 |
| 10 Sep 2025 (day-first) | 2025-09-10 |
| May 1 2026 (future date) | 2026-05-01 [data_flag = FUTURE_DATE] |

---

**2. SKU inconsistencies**

Python's `.str.strip().str.upper()` was applied to the `sku` column in both sheets. `strip()` removes any invisible leading or trailing whitespace that could cause join mismatches, and `upper()` standardizes all letters to uppercase. Applying the same transformation to both sheets ensures JOINs will now match correctly across all rows.

| Before (Raw) | After (Cleaned) |
|---|---|
| sku-c-1012 | SKU-C-1012 |
| sku-u-1013 | SKU-U-1013 |
| sku-u-1001 | SKU-U-1001 |
| sku-c-1020 | SKU-C-1020 |

---

**3. Country inconsistencies**

A Python dictionary map `{'USA':'US', 'Canada':'CA'}` was applied using pandas `.map()`, which replaces exact matches and leaves already-correct values unchanged. The 3 NULL rows were left NULL rather than guessed. The `ship_to` city field could be used to infer the correct country manually for those specific rows.

| Before (Raw) | After (Cleaned) |
|---|---|
| USA | US |
| Canada | CA |
| US | US |
| CA | CA |
| NULL | NULL (left for manual review) |

---

**4. Payment inconsistencies**

A canonical dictionary map was applied covering every observed variant: `'visa'` and `'VISA'` both map to `'Visa'`, `'MC'` maps to `'Mastercard'`, `'debit'` and `'Debit'` both map to `'Debit'`. Values not in the map were left as-is, producing 10 consistent title-case payment method labels.

| Before (Raw) | After (Cleaned) |
|---|---|
| visa | Visa |
| VISA | Visa |
| MC | Mastercard |
| debit | Debit |
| Mastercard | Mastercard |
| Apple Pay | Apple Pay |

---

**5. Quantity stored as string**

The `' units'` suffix was stripped using `str.replace(' units', '')` before casting to integer using `int(float(val))`. The `float()` intermediate step handles any cases where the number might have been stored with a decimal point.

| Before (Raw) | After (Cleaned) |
|---|---|
| 2 units (string) | 2 (integer) |
| 1 (integer) | 1 |
| 3 (integer) | 3 |

---

**6. Currency code embedded in unit_price**

Regex was used to strip the currency prefix before casting to float: `re.sub(r'(USD|CAD)\s*', '', str(val))` removes `'USD '` or `'CAD '` and any trailing space, leaving just the numeric portion. The currency code was simultaneously captured into a new `currency` column.

| Before (Raw) | After (Cleaned) |
|---|---|
| USD 18.99 (string) | unit_price = 18.99 \| currency = USD |
| CAD 46.99 (string) | unit_price = 46.99 \| currency = CAD |
| USD 89.99 (string) | unit_price = 89.99 \| currency = USD |

---

**7. Discount format inconsistencies**

A parsing function handled each pattern in sequence: percent strings had `'%'` stripped and were divided by 100; `'student 10%'` was caught by a keyword match on `'student'` and resolved to `0.10`; `'promo5'` was caught by a keyword match on `'promo'` with the trailing digit extracted as the rate; bare integers greater than 1 were divided by 100; zeros became `0.0`. NULLs were preserved rather than defaulted to 0.

| Before (Raw) | After (Cleaned) |
|---|---|
| 10% | 0.10 |
| student 10% | 0.10 |
| promo5 | 0.05 |
| 5 | 0.05 |
| 5% | 0.05 |
| 0 | 0.00 |
| NULL | NULL (preserved) |

---

**8. Tax format inconsistencies**

The `'HST'` label prefix was stripped first using `.replace('HST', '').strip()`. Percent strings were then detected by the presence of a trailing `'%'` and divided by 100. Values already in decimal form (less than 1) were left unchanged. NULLs were preserved — a missing tax value may indicate a tax-exempt line.

| Before (Raw) | After (Cleaned) |
|---|---|
| 7% | 0.07 |
| 8.25% | 0.0825 |
| 13% | 0.13 |
| 0.07 | 0.07 |
| 0.0825 | 0.0825 |
| HST 13% | 0.13 |
| NULL | NULL |

---

**9. Line total inconsistencies**

The `'$'` prefix was stripped using `str.replace('$', '')` before casting to float. The 62 NULL totals were left NULL because recalculation requires confirmed values for discount and tax, which are themselves sometimes NULL.

| Before (Raw) | After (Cleaned) |
|---|---|
| $19.43 (string) | 19.43 |
| $97.17 (string) | 97.17 |
| 161.97 (clean) | 161.97 |
| NULL | NULL |

---

**10. Return flag inconsistencies**

Existing `'Y'` and `'N'` values were normalized to uppercase using `.str.strip().str.upper()`. NULL values were filled with `'Unknown'` using `.fillna('Unknown')`, creating a three-value system that preserves the distinction between a confirmed non-return and a row where return status was never recorded.

| Before (Raw) | After (Cleaned) |
|---|---|
| NULL | Unknown |
| Y | Y |
| N | N |

---

**11. customer_info field**

A parsing function checked for each delimiter in priority order: pipe first, then semicolon, then slash. The string was split on whichever delimiter was found, and the first segment was stored as `customer_name`. The remaining segments were scanned for keywords: `'loyalty'` → Loyalty, `'student'` → Student, `'guest'` → Guest. Rows with no keyword defaulted to Standard.

| Before (Raw) | After (Cleaned) |
|---|---|
| Grace Hall \| Student \| US | customer_name = Grace Hall \| customer_type = Student |
| Mason Rivera; Loyalty? Y | customer_name = Mason Rivera \| customer_type = Loyalty |
| Zoe Garcia / guest order | customer_name = Zoe Garcia \| customer_type = Guest |
| Owen Clark | customer_name = Owen Clark \| customer_type = Standard |

---

**12. Incorrect email address**

Row LN-041 contained `'emma_wilson@school'`, which is missing its top-level domain. The value could not be automatically corrected since the intended domain is unknown. It was left in the cleaned file as-is and flagged for manual review.

| Before (Raw) | After (Cleaned) |
|---|---|
| emma_wilson@school (invalid) | emma_wilson@school (flagged for manual review) |

---

**13. Category conflated with customer type**

The `'/ Student'` suffix was removed from all sales category values using regex: `re.sub(r'\s*/\s*Student', '', val, flags=re.IGNORECASE)`. After cleaning, the 14 raw category variants collapsed to 7 clean categories: Tech, Accessories, School, Desk Setup, Apparel, Lifestyle, and Audio.

| Before (Raw) | After (Cleaned) |
|---|---|
| Accessories / Student | Accessories |
| Desk Setup / Student | Desk Setup |
| Tech / Student | Tech |
| Apparel / Student | Apparel |
| School / Student | School |
| Audio / Student | Audio |
| Tech | Tech |

---

### Product_Supplier_Master Fixes

**14. Duplicate SKU rows for product variants**

SKUs were standardized to uppercase using `.str.strip().str.upper()`. The duplicate rows were intentionally kept intact rather than collapsed, to preserve variant-specific data such as different weights, descriptions, and prices.

| Before (Raw) | After (Cleaned) |
|---|---|
| sku-c-1002 | SKU-C-1002 |
| sku-u-1007 | SKU-U-1007 |
| SKU-U-1005 | SKU-U-1005 |

---

**15. Reorder level inconsistencies**

A parsing function converted `'ten'` to the integer `10` via explicit string match. All other numeric string values were cast using `int(float(val))`. The 6 NULL values were left NULL as a missing reorder level is a business decision that cannot be inferred from the data.

| Before (Raw) | After (Cleaned) |
|---|---|
| ten | 10 (integer) |
| 5 (string) | 5 (integer) |
| 12 (string) | 12 (integer) |
| NULL | NULL |

---

**16. Vendor phone inconsistencies**

All non-digit characters were stripped using `re.sub(r'\D', '', val)`, leaving only the 10 raw digits. These were then reformatted into NXX-NXX-XXXX by slicing the digit string: `digits[:3] + '-' + digits[3:6] + '-' + digits[6:]`.

| Before (Raw) | After (Cleaned) |
|---|---|
| (404)555-0181 | 404-555-0181 |
| 604.555.0190 | 604-555-0190 |
| 647.555.0130 | 647-555-0130 |
| 416.555.0144 | 416-555-0144 |
| 404-555-0181 | 404-555-0181 |

---

**17. Vendor rep inconsistencies**

The `'/ email missing'` annotation was removed using `re.sub(r'\s*/\s*email missing', '', val, flags=re.IGNORECASE)`. The `'Ms. '` honorific prefix was removed with `str.replace('Ms. ', '')`. The typo `'Mia Dia zFernandez'` was corrected to `'Mia Diaz'` via direct string replacement.

| Before (Raw) | After (Cleaned) |
|---|---|
| Jason Wu / email missing | Jason Wu |
| Karan Singh / email missing | Karan Singh |
| Mia Dia zFernandez | Mia Diaz |
| Ms. Anika Roy | Anika Roy |
| Taylor Green | Taylor Green |

---

**18. Cost and list price inconsistencies**

The same regex stripping logic used for `unit_price` was applied: `re.sub(r'(USD|CAD)\s*', '', str(val))`. For rows with no prefix, the currency was inferred from the SKU — `-C-` maps to CAD, `-U-` maps to USD. The currency code was stored in a new `cost_currency` column.

| Before (Raw) | After (Cleaned) |
|---|---|
| USD 6.25 (string) | cost = 6.25 \| cost_currency = USD |
| CAD 89.99 (string) | cost = 89.99 \| cost_currency = CAD |
| 31.4 (bare number) | cost = 31.40 \| cost_currency = CAD (inferred from SKU-C-) |
| 27.8 (bare number) | cost = 27.80 \| cost_currency = CAD (inferred from SKU-C-) |

---

**19. Weight inconsistencies**

A parsing function used regex to extract the numeric portion with `re.search(r'[\d.]+', val)`. The unit was identified by keyword scanning: `'kg'`/`'kilogram'` → ×1000, `'lb'`/`'lbs'`/`'pound'` → ×453.592, `'oz'`/`'ounce'` → ×28.3495, `'g'` → no conversion. All results were rounded to two decimal places and stored in a new `weight_grams` column.

| Before (Raw) | After (Cleaned) |
|---|---|
| 8 ounces | 226.80 g |
| 8 oz | 226.80 g |
| 1.1 pound | 499.00 g |
| 1.1 lb | 499.00 g |
| 1.1 lbs | 499.00 g |
| 0.22 kilograms | 220.00 g |
| 0.22kg | 220.00 g |
| 340 grams | 340.00 g |
| 340 g | 340.00 g |
| 450g | 450.00 g |
| NULL | NULL |

---

**20. Length inconsistencies**

The numeric portion was extracted with `re.search(r'[\d.]+', val)`. Strings containing `'in'`, `'"'`, or `'inches'` were multiplied by 2.54 to convert to centimeters. Strings containing `'cm'` or `'centim'` were left as-is. All results were stored in a new `length_cm` column.

| Before (Raw) | After (Cleaned) |
|---|---|
| 10.2 in | 25.91 cm |
| 10.2" | 25.91 cm |
| 10.2 inches | 25.91 cm |
| 11.5 inches | 29.21 cm |
| 12.6 in | 32.00 cm |
| 25.4 cm | 25.40 cm |
| 25.4cm | 25.40 cm |
| 29.2 centimetres | 29.20 cm |
| 32cm | 32.00 cm |
| NULL | NULL |

---

**21. Category inconsistencies**

Comma and ampersand separators were replaced with `' / '` using `re.sub(r'[,&]\s*', ' / ', val)`. The entry `'Student and apparels'` was directly replaced with `'Apparel'`. `'Accessories / Accessories'` was collapsed to `'Accessories'`. Legitimate dual categorizations were left unchanged.

| Before (Raw) | After (Cleaned) |
|---|---|
| Lifestyle , Student | Lifestyle / Student |
| Tech & Student | Tech / Student |
| Student and apparels | Apparel |
| Accessories / Accessories | Accessories |
| Desk Setup / Accessories | Desk Setup / Accessories |

---

**22. Discontinued NULL values**

NULL values were filled with `'N'` using `.fillna('N')`, and the entire column was cast to uppercase string and stripped of whitespace. Staff are far more likely to flag a discontinued product than to leave an active one unflagged, making this a reasonable default — though it should be confirmed with the business owner.

| Before (Raw) | After (Cleaned) |
|---|---|
| NULL (17 rows) | N |
| Y | Y |
| N | N |


## Queries

### Required Query 1 — Which products generated the highest total sales revenue, by country?

```sql
SELECT o.ship_country, p.product_id, p.sku, p.product_description,
    SUM(ol.line_total) AS total_sales_revenue
FROM `Order` o
JOIN OrderLine ol ON o.order_id = ol.order_id
JOIN Product p ON ol.product_id = p.product_id
GROUP BY o.ship_country, p.product_id, p.sku, p.product_description
ORDER BY o.ship_country, total_sales_revenue DESC;
```
<img width="624" height="641" alt="Screenshot 2026-04-24 at 7 34 12 PM" src="https://github.com/user-attachments/assets/26728d46-784e-4fc1-87ea-f546377be31b" />

---

### Required Query 2 — Which employees handled the largest number of orders, and how do their results compare with other employees under the same manager?

```sql
SELECT emp.manager_id, emp.employee_id, emp.orders_handled,
    team.max_orders_under_manager, team.avg_orders_under_manager,
    emp.orders_handled - team.avg_orders_under_manager AS difference_from_manager_avg
FROM (
    SELECT e.manager_id, e.employee_id, COUNT(o.order_id) AS orders_handled
    FROM Employee e
    LEFT JOIN `Order` o ON e.employee_id = o.employee_id
    GROUP BY e.manager_id, e.employee_id
) 
JOIN (
    SELECT ec.manager_id,
        MAX(ec.orders_handled) AS max_orders_under_manager,
        AVG(ec.orders_handled) AS avg_orders_under_manager
    FROM (
        SELECT e.manager_id, e.employee_id, COUNT(o.order_id) AS orders_handled
        FROM Employee e
        LEFT JOIN `Order` o ON e.employee_id = o.employee_id
        GROUP BY e.manager_id, e.employee_id
    ) <img width="627" height="647" alt="Screenshot 2026-04-24 at 7 33 20 PM" src="https://github.com/user-attachments/assets/edb302c1-93f9-4b1b-8807-7283de0072aa" />

    GROUP BY ec.manager_id
) team ON emp.manager_id = team.manager_id
ORDER BY emp.orders_handled DESC, emp.manager_id, emp.employee_id;
```
<img width="1256" height="599" alt="Screenshot 2026-04-24 at 7 30 39 PM" src="https://github.com/user-attachments/assets/46e53a07-c84d-4e2f-bb3a-3032a88e3f96" />

---

### Required Query 3 — Which vendors supply products that appear in more than one category?

```sql
SELECT v.vendor_id, v.vendor_name,
    COUNT(DISTINCT p.category_id) AS number_of_categories
FROM Vendor v
JOIN Product p ON v.vendor_id = p.vendor_id
GROUP BY v.vendor_id, v.vendor_name
HAVING COUNT(DISTINCT p.category_id) > 1
ORDER BY number_of_categories DESC, v.vendor_name;
```
<img width="1257" height="432" alt="Screenshot 2026-04-24 at 7 31 00 PM" src="https://github.com/user-attachments/assets/63a5d566-13eb-4fd9-8692-618cc63b518b" />

---

### Additional Query 1 — Which products are frequently purchased together in the same order?

**Business justification:** This helps identify product bundling opportunities and cross-selling strategies by showing which products customers commonly purchase together in the same order. These insights can be used to improve marketing campaigns and product recommendations to increase overall sales.

```sql
SELECT ol1.product_id AS product_1_ID, ol2.product_id AS product_2_ID,
    COUNT(*) AS times_purchased_together
FROM OrderLine ol1
JOIN OrderLine ol2
    ON ol1.order_id = ol2.order_id
    AND ol1.product_id < ol2.product_id
GROUP BY ol1.product_id, ol2.product_id
ORDER BY times_purchased_together DESC;
```
<img width="1130" height="496" alt="Screenshot 2026-04-24 at 7 34 44 PM" src="https://github.com/user-attachments/assets/638ec917-5b05-460d-bd7b-b2d14fc2dfad" />

---

### Additional Query 2 — Which products have high sales volume but also high return counts?

**Business justification:** This helps identify popular products that may also have quality, sizing, or customer satisfaction issues, allowing the company to address potential problems and reduce return rates. The results can guide improvements in product design, descriptions, or supplier selection.

```sql
SELECT p.product_id, p.sku, p.product_description,
    SUM(ol.quantity) AS total_units_sold,
    SUM(CASE WHEN ol.return_flag = 'Y' THEN ol.quantity ELSE 0 END) AS total_units_returned,
    ROUND(SUM(CASE WHEN ol.return_flag = 'Y' THEN ol.quantity ELSE 0 END) * 100.0
        / SUM(ol.quantity), 2) AS return_rate_percent,
    SUM(ol.line_total) AS total_revenue
FROM Product p
JOIN OrderLine ol ON p.product_id = ol.product_id
GROUP BY p.product_id, p.sku, p.product_description
HAVING SUM(ol.quantity) > 0
ORDER BY return_rate_percent DESC, total_units_sold DESC;
```
<img width="1131" height="498" alt="Screenshot 2026-04-24 at 7 35 04 PM" src="https://github.com/user-attachments/assets/c8b8905a-a0f7-4530-815f-ec7b2c9ed2e9" />

---

### Additional Query 3 — Which vendors generate the most revenue, and how many different categories do they supply?

**Business justification:** This helps Northline identify its most valuable suppliers and determine whether top revenue is driven by vendors with a wide range of product categories or more specialized offerings.

```sql
SELECT v.vendor_id, v.vendor_name,
    COUNT(DISTINCT p.product_id) AS number_of_products,
    COUNT(DISTINCT p.category_id) AS number_of_categories,
    SUM(ol.line_total) AS total_vendor_revenue
FROM Vendor v
JOIN Product p ON v.vendor_id = p.vendor_id
JOIN OrderLine ol ON p.product_id = ol.product_id
GROUP BY v.vendor_id, v.vendor_name
ORDER BY total_vendor_revenue DESC;
```
<img width="1125" height="383" alt="Screenshot 2026-04-24 at 7 35 22 PM" src="https://github.com/user-attachments/assets/1e71feb9-1ff1-4415-879d-2e4afa80aba0" />

## Databse Information
Name of the database: mb_B5
Additional information: Each query listed above is marked in the database using stored procedures which can be called using the following format: CALL mbB5_Q1();
