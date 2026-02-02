# E-commerce Growth & Retention Command Center (Olist)

## Objetivo
Construir un dashboard ejecutivo y operativo para entender crecimiento, retención, experiencia de entrega y satisfacción del cliente.

## Dataset
Olist Brazilian E-Commerce (Kaggle). Fuente: Kaggle (descarga manual).

## Preguntas de negocio (MVP)
1) ¿Cómo evolucionan pedidos, GMV/ventas y AOV en el tiempo?
2) ¿Qué cohortes retienen mejor y peor? (retención por mes de primera compra)
3) ¿Dónde se está rompiendo la experiencia logística? (entrega a tiempo, demoras)
4) ¿Qué categorías/sellers impulsan crecimiento y cuáles generan problemas (cancelaciones/reviews)?

## KPIs (definiciones iniciales)
- Orders: cantidad de pedidos (order_id)
- Customers: clientes únicos (customer_unique_id)
- GMV: suma de payment_value
- AOV: GMV / Orders
- Repeat rate: % clientes con >=2 pedidos
- On-time delivery rate: % entregas en o antes de fecha estimada
- Avg delivery delay (days): delivered - estimated

## Páginas del reporte (MVP)
1) Executive Overview
2) Growth
3) Retention (Cohorts + RFM)
4) Logistics
5) Reviews / Satisfaction

## Entregables en GitHub
- README con contexto + preguntas + hallazgos
- Capturas del dashboard (images/)
- SQL (staging/marts + checks)
- Diccionario de métricas (docs/01_metrics.md)
- Diagrama del modelo (docs/02_model.png)
