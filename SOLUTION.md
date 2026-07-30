
# BrightCart E-Commerce Profitability Analysis (MySQL Guide)

This guide provides a complete, step-by-step walkthrough of BrightCart's profitability analysis using MySQL. It connects order-level transaction data, product cost structures, and channel-level marketing spend to uncover true net profit margins and pinpoint exact areas of margin erosion.

---

## 📋 Table of Contents
1. [Data Model Overview](#1-data-model-overview)
2. [Step 1: Data Preparation & Base Views](#step-1-data-preparation--base-views)
3. [Step 2: Item-Level Gross Margin Analysis](#step-2-item-level-gross-margin-analysis)
4. [Step 3: Order-Level Net Profitability & Shipping Erosion](#step-3-order-level-net-profitability--shipping-erosion)
5. [Step 4: Marketing Attribution & Blended CAC](#step-4-marketing-attribution--blended-cac)
6. [Step 5: Executive Profitability Summary & Erosion Diagnosis](#step-5-executive-profitability-summary--erosion-diagnosis)
7. [Key Business Takeaways](#key-business-takeaways)

---

## Data Model Overview

The analysis relies on five core tables in the MySQL database:

| Table | Description |
| :--- | :--- |
| `orders` | Transaction details (`order_id`, `customer_id`, `order_date`, `shipping_cost_charged`, `shipping_cost_actual`) |
| `products` | Product catalog & costs (`product_id`, `category`, `product_name`, `cogs`) |
| `marketing_spend` | Ad spend tracking (`campaign_id`, `channel`, `spend_date`, `spend_amount`) |


---

## 1: Database & Schema Setup
Execute the DDL below to set up tables in MySQL and import the datasets.

```sql
CREATE DATABASE IF NOT EXISTS brightcart_db;
USE brightcart_db;

-- 1. Orders Table DDL
CREATE TABLE IF NOT EXISTS orders (
    order_id VARCHAR(20) PRIMARY KEY,
    customer_id VARCHAR(20),
    order_date DATE,
    channel VARCHAR(50),
    payment_method VARCHAR(50),
    region VARCHAR(50),
    items_ordered INT,
    primary_category VARCHAR(50),
    gross_revenue DECIMAL(10,2),
    discount_pct INT,
    discount_amount DECIMAL(10,2),
    shipping_cost DECIMAL(10,2),
    product_cost DECIMAL(10,2),
    platform_fee DECIMAL(10,2),
    transaction_fee DECIMAL(10,2),
    returned VARCHAR(5),
    refund_amount DECIMAL(10,2),
    net_revenue DECIMAL(10,2),
    total_costs DECIMAL(10,2),
    profit DECIMAL(10,2)
);

-- 2. Products Table DDL
CREATE TABLE IF NOT EXISTS products (
    product_id VARCHAR(20) PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    sub_category VARCHAR(50),
    unit_cost DECIMAL(10,2),
    selling_price DECIMAL(10,2),
    shipping_cost_per_unit DECIMAL(10,2),
    weight_lbs DECIMAL(8,2),
    supplier VARCHAR(100)
);

-- 3. Marketing Spend Table DDL
CREATE TABLE IF NOT EXISTS marketing_spend (
    month VARCHAR(7),
    platform VARCHAR(50),
    spend DECIMAL(10,2),
    impressions INT,
    clicks INT,
    conversions INT,
    revenue_attributed DECIMAL(10,2),
    cpc DECIMAL(10,2),
    cpa DECIMAL(10,2),
    roas DECIMAL(10,2)
);

## Step 1: Data Integrity & Verification
Before running aggregations, verify that cost, revenue, and profit metrics align with standard formula logic across all order rows:$$\text{Total Costs} = \text{Product Cost} + \text{Shipping Cost} + \text{Platform Fee} + \text{Transaction Fee}$$$$\text{Net Revenue} = \text{Gross Revenue} - \text{Discount Amount} - \text{Refund Amount}$$$$\text{Profit} = \text{Net Revenue} - \text{Total Costs}$$
