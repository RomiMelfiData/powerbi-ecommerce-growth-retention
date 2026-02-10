# DAX Measures Catalog

Tabla de medidas principales del reporte, con definición y uso.

| Measure | Qué calcula | Dónde se usa |
|---|---|---|
| Orders | Cantidad de pedidos (distinctcount order_id) | Executive Overview / Cards |
| Customers (unique) | Clientes únicos (distinctcount customer_unique_id) | Executive Overview / Cards |
| GMV | Suma de fact_orders[payment_total] | Todas las páginas |
| AOV | GMV / Orders | Executive Overview / Cards |
| Delivered Orders | Pedidos entregados | Executive Overview / SLA |
| On-time Rate | % entregas a tiempo | Executive Overview / SLA |
| Avg Review Score | Promedio score de reviews | Overview / Quality |
| Returning Rate | % clientes que vuelven a comprar al menos una vez | Retención (Cohortes) |
| Retention % | Retención por mes desde cohorte (MonthsSinceCohort) | Matriz de cohortes |
| Recency (days) | Días desde última compra (vs fecha máxima del dataset) | Customer Segments |
| Frequency (orders) | Pedidos por cliente | Customer Segments |
| Monetary (GMV) | GMV por cliente | Customer Segments |
| Customers in Recency Band | Clientes por banda de recencia | Customer Segments |
| GMV in Recency Band | GMV por banda de recencia | Customer Segments |
| GMV Share in Band | % del GMV total aportado por la banda seleccionada | Customer Segments |
