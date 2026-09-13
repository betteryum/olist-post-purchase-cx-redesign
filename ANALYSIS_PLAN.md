# Analysis Plan

## Analysis Unit
Order-level analysis, using delivered orders with valid delivery timestamps.

## Core Master Table
The project will create a master order table with:
- order_id
- customer_unique_id
- seller_id
- product_category_name_english
- customer_state
- seller_state
- order_purchase_timestamp
- order_approved_at
- order_delivered_carrier_date
- order_delivered_customer_date
- order_estimated_delivery_date
- delay_days
- is_late
- review_score
- is_bad_review
- review_comment_message
- price
- freight_value

## Analysis Modules

### 1. Data Understanding
Goal:
Understand tables, keys, missing values, and join logic.

### 2. Delivery Failure Diagnosis
Goal:
Calculate late delivery rate, delay days, and delay severity.

### 3. Review Score Impact
Goal:
Compare review scores and bad review rates between late and on-time orders.

### 4. Seller / Category / Region Risk
Goal:
Identify where operational and CX risk is concentrated.

### 5. Retention
Goal:
Compare repeat purchase behavior after late vs on-time first orders.

### 6. Review Text Coding
Goal:
Translate and code low-score reviews into pain point categories.

### 7. Service Design Synthesis
Goal:
Turn data findings into journey map, service blueprint, opportunity matrix, and service recovery concept.
