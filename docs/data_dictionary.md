# Data Dictionary

## Modelo (esquema estrella)
**Fact tables**
- **fact_orders** — grano: 1 fila por pedido (order_id)
- **fact_sales** — grano: 1 fila por ítem de pedido (order_item_id / order_id + product_id)

**Dimensions**
- **dim_date** — 1 fila por día
- **dim_customers** — 1 fila por cliente (customer_id / customer_unique_id)
- **dim_products** — 1 fila por producto (product_id)
- **dim_sellers** — 1 fila por vendedor (seller_id)

## Tablas y campos clave

### fact_orders (pedido)
**Key:** order_id  
Campos principales:
- customer_id
- customer_unique_id
- purchase_date
- order_status
- payment_total
- delivered_date
- estimated_date
- is_delivered
- on_time_flag
- delivery_delay_days
- CohortMonth
- MonthsSinceCohort

### fact_sales (ítems del pedido)
**Key:** order_item_id (y order_id)  
Campos principales:
- order_id
- customer_unique_id
- product_id
- seller_id
- purchase_date
- item_total
- freight_value
- price
- order_status

### dim_date
**Key:** Date  
Campos principales:
- Date
- Year
- MonthNum
- YearMonth

### dim_customers
**Key:** customer_id / customer_unique_id  
Campos principales:
- customer_unique_id
- customer_city
- customer_state
- customer_zip_code_prefix

### dim_products
**Key:** product_id  
Campos principales:
- product_id
- product_category_name
- product_category_name_english
- category_final

### dim_sellers
**Key:** seller_id  
Campos principales:
- seller_id
- seller_city
- seller_state
- seller_zip_code_prefix

## Relaciones principales
- fact_orders[purchase_date] → dim_date[Date]
- fact_orders[customer_id] → dim_customers[customer_id]
- fact_sales[purchase_date] → dim_date[Date]
- fact_sales[customer_id] → dim_customers[customer_id]
- fact_sales[product_id] → dim_products[product_id]
- fact_sales[seller_id] → dim_sellers[seller_id]
