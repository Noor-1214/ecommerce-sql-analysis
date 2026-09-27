
# E-Commerce SQL Analysis
An end-to-end SQL analysis of ~100,000 orders from Olist, a Brazilian e-commerce marketplace. The project moves from data loading and validation through to business questions on sales, customers, delivery performance, customer satisfaction and payments.

## Business questions

1. Which product categories generate the most revenue?
2. How many customers come back to buy again?
3. Which regions experience the most delivery delays?
4. How satisfied are customers, and what drives dissatisfaction?
5. How do customers pay, and what does a typical order look like?

## Dataset

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle), covering orders from 2016 to 2018. Monetary values are in Brazilian reais (R$).

| Table | Rows | Description |
|---|---|---|
| customers | 99,441 | Customer records and location |
| orders | 99,441 | Order status and timestamps |
| order_items | 112,650 | Products within each order |
| products | 32,951 | Product attributes and category |
| category_translation | 71 | Portuguese → English category names |
| order_reviews | 99,224 | Review scores (1–5) |
| order_payments | 103,886 | Payment method, value and instalments |

## Tools and skills

- **SQLite** with **DB Browser for SQLite**
- Multi-table `JOIN`s (up to three tables)
- Aggregation with `GROUP BY` and `HAVING`
- Common Table Expressions (`WITH`)
- Window functions (`SUM() OVER ()`, `ROW_NUMBER()`, `COUNT() OVER ()`)
- Conditional logic with `CASE WHEN`
- Date calculations (`julianday`, `DATE`, `SUBSTR`)
- Data validation and data-quality checks

## Key findings

### 1. Late deliveries are strongly associated with poor reviews

| Delivery status | Reviews | Avg review score | % negative (1–2★) |
|---|---|---|---|
| Late | 6,409 | 2.27 | 62.41% |
| On time | 89,944 | 4.29 | 9.28% |

Late orders make up only ~6.7% of delivered orders, but they account for roughly a third of all negative reviews. Late orders are almost 7× more likely to receive a 1–2 star review.

### 2. Customer satisfaction is high overall but polarised

| Score | Reviews | % |
|---|---|---|
| 1★ | 11,424 | 11.51% |
| 2★ | 3,151 | 3.18% |
| 3★ | 8,179 | 8.24% |
| 4★ | 19,142 | 19.29% |
| 5★ | 57,328 | 57.78% |

77% of reviews are 4 or 5 stars. However, 1-star reviews are over three times as common as 2-star reviews: when customers are unhappy, they tend to be very unhappy, which is consistent with clear service failures such as late deliveries.

### 3. Delivery performance varies widely by state

States with at least 500 delivered orders, ranked by late-delivery rate:

| State | Orders | Avg delivery (days) | Late rate |
|---|---|---|---|
| MA | 717 | 21.57 | 17.43% |
| CE | 1,279 | 21.27 | 13.76% |
| BA | 3,256 | 19.34 | 12.16% |
| RJ | 12,350 | 15.31 | 12.11% |
| PA | 946 | 23.77 | 11.21% |

Slow delivery and late delivery are not the same thing. PA has the longest average delivery time but a lower late rate than MA, suggesting generous delivery estimates. RJ has a relatively short delivery time but a high late rate, suggesting its estimates are too optimistic. Because RJ has by far the highest order volume in this group, it represents the largest operational risk.

### 4. Very few customers return

| Orders placed | Customers | % |
|---|---|---|
| 1 | 90,557 | 97.00% |
| 2 | 2,573 | 2.76% |
| 3 | 181 | 0.19% |
| 4+ | 47 | 0.05% |

Only **2,801 of 93,358 customers (3.0%)** made more than one delivered purchase. Olist's revenue depends almost entirely on acquiring new customers.

### 5. Top product categories by revenue

| Category | Product revenue (R$) |
|---|---|
| health_beauty | 1,285,681.34 |
| watches_gifts | 1,205,005.68 |
| bed_bath_table | 1,036,988.68 |
| sports_leisure | 988,048.97 |
| computers_accessories | 911,954.32 |

### 6. Credit card dominates payments

| Payment type | Orders | Total value (R$) | Avg payment (R$) | % of value |
|---|---|---|---|---|
| credit_card | 76,505 | 12,542,084.19 | 163.32 | 78.34% |
| boleto | 19,784 | 2,869,361.27 | 145.03 | 17.92% |
| voucher | 3,866 | 379,436.87 | 65.70 | 2.37% |
| debit_card | 1,528 | 217,989.79 | 142.57 | 1.36% |

*Boleto* is a Brazilian payment method in which the customer pays a bank slip in cash or via online banking. Vouchers have a low average because they are typically used for part of an order alongside another method.

### 7. The typical order is smaller than the average suggests

| Metric | Value (R$) |
|---|---|
| Mean order value | 159.86 |
| Median order value | 105.28 |
| Minimum | 9.59 |
| Maximum | 13,664.08 |

Order values are heavily right-skewed, so the median is the better measure of a typical order. SQLite has no built-in median function, so it was calculated with `ROW_NUMBER()` and `COUNT() OVER ()`.

## Recommendations

1. **Improve delivery estimates and logistics in high-risk states**, starting with RJ (highest volume) and the North-East (MA, CE, BA). Reducing late deliveries is the clearest lever for improving customer satisfaction.
2. **Communicate proactively when orders are running late.** Since late orders drive a large share of negative reviews, early updates or small goodwill gestures could limit the damage.
3. **Invest in retention.** With a 3% repeat rate, even a small improvement through post-purchase follow-up, loyalty offers or better delivery experiences would have a meaningful effect on revenue.

## Data-quality notes and definitions

- **Customer identity:** `customer_id` is generated per order, so it cannot identify repeat customers (using it returns 0 repeat buyers). `customer_unique_id` was used for all customer-level analysis.
- **Late delivery:** an order is counted as late only if it was delivered on a *later calendar date* than estimated. Estimated delivery dates are stored as midnight, so a timestamp comparison would wrongly count same-day deliveries as late.
- **Revenue definitions:** category revenue uses `order_items.price` (product value, excluding freight). Payment analysis and order values use `order_payments.payment_value`, which includes freight.
- **Category coverage:** products with no category, or with a category missing from the translation table, are excluded from the category revenue analysis by the inner join.
- **Delivered orders only:** customer, delivery, review and order-value analyses are limited to orders with status `delivered`.
- **Minimum volume:** the state analysis only includes states with at least 500 delivered orders, to avoid misleading rates from small samples.
- **Payments:** 3 payment records have type `not_defined` and a value of R$0. One delivered order has no matching payment record, so order-value analysis covers 96,477 orders.
- **Correlation, not causation:** the link between late delivery and low review scores is observational; it shows a strong association rather than proving cause and effect.

