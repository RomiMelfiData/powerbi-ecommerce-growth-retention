# E-commerce Growth & Retention Dashboard (Olist)

## Objetivo
Construir un dashboard ejecutivo y analítico en Power BI para entender el desempeño de un e-commerce de punta a punta, con foco en:
**crecimiento (GMV/Orders/AOV)**, **mix de categorías**, **retención por cohortes** y **segmentación de clientes (RFM)**.

## Dataset
**Olist Brazilian E-Commerce Public Dataset (Kaggle)** (descarga manual).  
Los datos crudos no se suben al repositorio (ver `data_raw/` en `.gitignore`).

## Alcance (qué incluye el proyecto)
- Modelado en **esquema estrella** (facts + dims).
- Transformaciones en **Power Query** y cálculo de medidas en **DAX**.
- Análisis por período (filtros de fecha) y por categoría.
- Retención basada en cohortes (mes de primera compra) y meses desde la cohorte.
- Segmentación RFM orientada a priorización comercial (win-back vs fidelización).

## Preguntas de negocio (MVP)
1) ¿Cómo evolucionan **Orders**, **GMV** y **AOV** mes a mes?
2) ¿Qué categorías explican el volumen y cómo cambia el **mix** en el tiempo?
3) ¿Qué tan alta/baja es la **recompra** y cómo se comporta la **retención por cohortes**?
4) ¿Cómo se distribuyen los clientes por **recencia** y dónde está el **valor (GMV)** por segmento?

## KPIs (definiciones)
- **Orders:** cantidad de pedidos (DISTINCTCOUNT `order_id`)
- **Customers (unique):** clientes únicos (DISTINCTCOUNT `customer_unique_id`)
- **GMV:** suma de `fact_orders[payment_total]`
- **AOV:** GMV / Orders
- **Returning Rate:** % de clientes que compran al menos una vez después de su primera compra (cohortes)
- **Retention % (Cohorts):** retención por `MonthsSinceCohort` (M0, M1, M2…)

## Páginas del reporte
1) **Executive Overview**
   - KPIs principales, tendencia temporal, lectura rápida por categoría.
2) **Growth Drivers & Category Mix**
   - Drivers de GMV y cambios del mix Top categorías en el tiempo.
3) **Retención y Recompra (Cohortes)**
   - Matriz de cohortes y Returning Rate.
4) **Segmentación de Clientes (RFM)**
   - Top customers, recency segments, GMV por segmento.


