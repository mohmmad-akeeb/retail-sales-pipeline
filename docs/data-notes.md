# Olist Dataset Notes

## Dataset Overview

Dataset:
Olist Brazilian E-Commerce

Source:
Kaggle

Source tables:
9

Approximate order count:
~100K

Date range:
[To be measured]

---

## 1. customers

### Row count
[Measure]

### Columns
[Record]

### Important keys
[Record]

### Grain
[Explain]

---

## 2. orders

### Row count
[Measure]

### Columns
[Record]

### Important keys
[Record]

### Grain
[Explain]

### order_id uniqueness
[Measure]

### Order status distribution
[Measure]

### Order date range
[Measure]

### Orders by month
[Measure]

---

## 3. order_items

### Row count
[Measure]

### Grain
[Determine]

### Important keys
[Determine]

---

## 4. order_payments

### Row count
[Measure]

### Grain
[Determine]

### Important keys
[Determine]

---

## 5. order_reviews

### Row count
[Measure]

### Grain
[Determine]

### Is review_id unique?
[Determine]

### Important keys
[Determine]

---

## 6. products

### Row count
[Measure]

### Important keys
[Determine]

---

## 7. sellers

### Row count
[Measure]

### Important keys
[Determine]

---

## 8. geolocation

### Row count
[Measure]

### Important keys
[Determine]

---

## 9. product_category_name_translation

### Row count
[Measure]

### Important keys
[Determine]

---

# Important Relationships

## customer_id vs customer_unique_id

customer_id count:
[Measure]

customer_unique_id count:
[Measure]

Gap:
[Calculate]

Meaning:
[Explain in my own words]

---

## Order → Items

Relationship:
[Determine]

Grain of order_items:
[Determine]

---

## Order → Payments

Relationship:
[Determine]

Grain of order_payments:
[Determine]

---

## Order → Reviews

Relationship:
[Determine]

Grain of order_reviews:
[Determine]

---

## Fan-out experiment

Naive orders → items → payments join row count:
[Measure]

order_items row count:
[Measure]

Difference:
[Explain]

Why this matters:
[Explain]

---

## Delivery dates

Null delivery-date counts by order status:
[Measure]

Observation:
[Explain]

---

# Surprises / Quirks

- [Finding]
- [Finding]
- [Finding]