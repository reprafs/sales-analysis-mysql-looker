# sales-analysis-mysql-looker
SQL-based sales analysis: monthly trends, top products, customer spending, and a dashboard in Looker Studio.

This project analyzes sales data using **MySQL queries** and **Looker Studio** for the dashboard.

The data has 4 tables: `customers`, `products`, `orders`, and `sales`.

---

## Setup

1. Open XAMPP (`manager-osx`).
2. Start **MySQL Database** and **Apache Web Server**.
3. Open http://localhost/phpmyadmin/
4. Create a new database named `project_db`.
5. Import the 4 files (the first row of each file is the header).
6. Set the **primary keys** and **foreign keys** for each table.
7. Open the **Designer** tab in phpMyAdmin and create the relationships between tables.
8. Change the date columns from `VARCHAR` to `DATE`:

```sql
ALTER TABLE `orders` CHANGE `order_date` `order_date` DATE NULL DEFAULT NULL;
ALTER TABLE `orders` CHANGE `delivery_date` `delivery_date` DATE NULL DEFAULT NULL;
```

---

## Data Cleaning

Before analyzing, I checked the data quality.

**1. Duplicate customers** (same name and address)

```sql
SELECT customer_name, address, COUNT(*) AS duplicate_count
FROM customers
GROUP BY customer_name, address
HAVING COUNT(*) > 1;
```

**2. NULL and empty values**

```sql
SELECT *
FROM customers
WHERE customer_name IS NULL OR TRIM(customer_name) = '';

SELECT *
FROM products
WHERE product_name IS NULL
   OR TRIM(product_name) = ''
   OR price IS NULL
   OR quantity IS NULL;

SELECT *
FROM sales
WHERE product_id IS NULL
   OR quantity IS NULL
   OR price_per_unit IS NULL
   OR total_price IS NULL;
```

**3. Check sales calculation** (`price_per_unit × quantity` should equal `total_price`)

```sql
SELECT COUNT(*) AS mismatch_count
FROM sales
WHERE price_per_unit * quantity <> total_price;
```

**4. Compare sales price with product price**

```sql
SELECT COUNT(*) AS mismatch_count
FROM sales s
JOIN products p
    ON s.product_id = p.product_ID
WHERE s.price_per_unit <> p.price;
```

**5. Check value ranges** (to find strange values)

```sql
SELECT
    (SELECT MIN(age) FROM customers) AS min_age,
    (SELECT MAX(age) FROM customers) AS max_age,
    (SELECT MIN(price) FROM products) AS min_price,
    (SELECT MAX(price) FROM products) AS max_price,
    (SELECT MIN(quantity) FROM products) AS min_quantity,
    (SELECT MAX(quantity) FROM products) AS max_quantity,
    (SELECT MIN(order_date) FROM orders) AS earliest_order,
    (SELECT MAX(order_date) FROM orders) AS latest_order;
```

**6. Check country and state**

```sql
SELECT country, COUNT(DISTINCT state) AS jumlah_state, COUNT(*) AS jumlah_customer
FROM customers
GROUP BY country;
```

Result: there is only **1 country** and **8 states**.

---

## Business Questions

1. How is our sales performance over this period? Are there time patterns we should watch?
2. Which products are the best sellers? Is any category falling behind?
3. How much do our customers spend?
4. Which region is the main market, and which one is still weak?
5. Who are our customers, and which age group matters most?
6. How many registered customers actually buy something?
7. Does the pattern change by month, product, or region?

## Analysis Decisions

- Revenue = `SUM(sales.total_price)`, not payment.
- Gender is not merged into "Other".
- The sample size per gender is small (about 100), so don't over-interpret it.

---

## Analysis

### 1. Orders, quantity sold, and revenue per month

![Query](images/photo1.png)

Result:

![Result](images/photo2.png)

### 2. Best-selling product type (quantity and revenue)

![Query](images/photo3.png)

Result:

![Result](images/photo4.png)

### 3. AOV per order, average spending per customer, and average quantity

![Query](images/photo5.png)

Result:

![Result](images/photo6.png)

### 4. Average delivery time per state

This one may not be used in the final dashboard.

![Query](images/photo7.png)

Result:

![Result](images/photo7.png)

### 5. Customers and revenue per state

![Query](images/photo8.png)

Result:

![Result](images/photo9.png)

### 6. Active customers who order more than once, and their revenue contribution

![Query](images/photo10.png)

Result:

![Result](images/photo11.png)

### 7. Revenue per age group (target market)

![Query](images/photo12.png)

Result:

![Result](images/photo13.png)

---

## Dashboard

Made with Looker Studio.

![Dashboard](images/photo14.png)

---

## Insights

> Fill in with your numbers.

1. **Monthly sales:** ...
2. **Products:** ...
3. **Customer spending:** ...
4. **Region:** ...
5. **Age group:** ...
6. **Repeat customers:** ...

## Recommendations

> Fill in with your numbers.

1. ...
2. ...
3. ...
4. ...
5. ...
