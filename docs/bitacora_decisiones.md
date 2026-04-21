# Bitácora de decisiones

## Etapa: Preparación de datos

**Objetivo general de esta etapa**
Construir una base de datos confiable para el dashboard, evitando duplicaciones y dejando el modelo listo para calcular KPIs correctamente (ventas/GMV, AOV, logística y retención).

---

### agg_payments_order (Pagos agregados por pedido)

**Problema identificado**
La tabla `stg_payments` puede tener más de una fila por `order_id` porque un pedido puede pagarse en cuotas o con múltiples métodos de pago. Si se combina `stg_payments` directamente con `stg_orders`, el resultado genera duplicación de pedidos (múltiples filas para un mismo `order_id`).

**Transformación aplicada**
Agrupar por `order_id` y calcular:
- `payment_total` = Suma de `payment_value` (total pagado por pedido)
- `payment_count` = Recuento de filas (cantidad de registros de pago del pedido)

**Finalidad**
Garantizar que el total pagado por pedido esté disponible sin duplicar pedidos y que métricas como Orders, GMV y AOV sean correctas.

---

### fact_orders (Tabla de hechos a nivel pedido)

**Problema identificado**
La tabla principal debe estar a nivel pedido (order) para calcular correctamente: cantidad de pedidos, GMV total, ticket promedio (AOV), métricas logísticas y retención por cohorte.

**Transformaciones aplicadas**

1. **Columnas de fecha (sin hora)**
   - `purchase_date` = `Date.From(order_purchase_timestamp)`
   - `delivered_date` = `Date.From(order_delivered_customer_date)`
   - `estimated_date` = `Date.From(order_estimated_delivery_date)`
   - *Motivo:* facilita el análisis temporal y la conexión con la tabla calendario.

2. **Indicadores logísticos**
   - `is_delivered`: identifica pedidos entregados
   - `delivery_delay_days`: diferencia en días entre entrega real y fecha estimada
   - `on_time_flag`: 1 si llegó a tiempo, 0 si llegó tarde
   - *Motivo:* habilita KPIs de SLA/logística y análisis de demoras.

3. **Merge con agg_payments_order** — join Externa izquierda por `order_id` para incorporar `payment_total` y `payment_count`.
   - *Motivo:* sumar el valor pagado por pedido dentro de `fact_orders` manteniendo 1 fila por pedido.

**Resultado**
- `agg_payments_order` asegura pagos a nivel pedido sin duplicaciones.
- `fact_orders` queda lista para KPIs ejecutivos, logística y retención.

---

## Etapa: Reviews

### Reviews agregadas por pedido + Merge a fact_orders

**Objetivo**
Incorporar la satisfacción del cliente al nivel de pedido para habilitar métricas como score promedio, % de pedidos con review y segmentaciones sin duplicar pedidos.

**Problema identificado**
`stg_reviews` tiene identificadores propios (`review_id`) y puede incluir múltiples registros por pedido. En un primer intento se agrupó por `review_id`, lo que generó una tabla sin `order_id` que no se podía combinar con `fact_orders`.

**Decisión tomada**
Crear `agg_reviews_order` con 1 fila por `order_id` para garantizar compatibilidad con `fact_orders`.

**Transformación aplicada**
1. Crear `agg_reviews_order` como referencia de `stg_reviews`.
2. Agrupar por `order_id`:
   - `review_score_avg` = Promedio de `review_score`
   - `review_count` = Recuento de filas

**Merge aplicado**
Join Externa izquierda entre `fact_orders` y `agg_reviews_order` por `order_id`. Se expandieron `review_score_avg` y `review_count`.

**Finalidad**
- Mantener `fact_orders` con 1 fila por pedido y sumar información de satisfacción sin inflar métricas.
- Habilitar análisis de satisfacción por fecha, estado del pedido, logística y retención.

---

## Etapa: Ventas por ítem

### Construcción de fact_sales (tabla de hechos a nivel ítem)

**Objetivo**
Crear una tabla de hechos para analizar ventas por producto, seller y categoría, manteniendo `order_id` y evitando duplicaciones al calcular métricas.

**Decisión de granularidad**
`fact_sales`: 1 fila = 1 ítem vendido (`order_id` + `order_item_id`)

**Motivo**
El análisis por producto/seller requiere el detalle de cada ítem. Con solo una tabla a nivel pedido se pierde granularidad para ranking de productos, ventas por categoría, costos de envío por ítem y mix de productos.

**Enriquecimiento con datos del pedido**

*Problema:* `stg_order_items` no contiene `customer_id` ni fecha de compra.

*Decisión:* Combinar `fact_sales` con `stg_orders` por `order_id` (join Externa izquierda) para traer:
- `customer_id`
- `order_purchase_timestamp`
- `order_status`

**Normalización temporal**
- Se creó `purchase_date` (tipo Date) a partir de `order_purchase_timestamp` (DateTime).
- Se eliminó `order_purchase_timestamp` en `fact_sales` para mantener el modelo limpio.

**Métrica base por ítem**
- `item_total` = `price` + `freight_value`
- *Motivo:* facilita reportes de venta total por ítem y comparativas por producto/seller/categoría.

**Resultado**
- `fact_sales` definida como tabla de hechos a nivel ítem.
- Se incorporaron `customer_id` y `purchase_date` para segmentación y análisis temporal.
- Se creó `item_total` como métrica operativa.
- El modelo mantiene **grano claro** para evitar duplicaciones.

---

## Dimensiones

### dim_customers, dim_products, dim_sellers y traducción de categorías

**Objetivo**
Construir dimensiones limpias para un modelo estrella estable.

**Decisiones principales**
- Mantener solo dimensiones y hechos en el modelo final.
- Dimensiones con 1 fila por entidad (clave única) para evitar relaciones many-to-many.

**dim_customers**
- Construida desde `stg_customers`.
- Columnas MVP: `customer_id`, `customer_unique_id`, ubicación.
- Se aplicó "Quitar duplicados" por `customer_id` para asegurar cardinalidad 1:*.

**dim_products con traducción**
- Se creó `stg_category_translation` desde `product_category_name_translation.csv`.
- Se combinó `dim_products` con `stg_category_translation` para incorporar `product_category_name_english`.
- Se quitaron duplicados por `product_id`.

**dim_sellers**
- Construida desde `stg_sellers`.
- Se mantuvieron columnas de identificación y ubicación, sin duplicados por `seller_id`.

**Finalidad**
- Las dimensiones permiten segmentar métricas sin duplicaciones y con un modelo escalable.
- La traducción de categorías mejora la claridad del dashboard para usuarios no portugueses.

---

### Creación de dim_date y corrección de tipos de fecha

**Objetivo**
Contar con una tabla calendario oficial para análisis temporal, time intelligence y segmentación por mes/año.

**Problema identificado**
Al crear `dim_date` con `CALENDAR(MIN, MAX)` se produjo error porque `purchase_date` estaba como texto (ABC), lo que alteraba el cálculo de MIN/MAX.

**Acción correctiva**
- Se corrigió `purchase_date` en Power Query para que sea tipo Fecha en `fact_orders` y `fact_sales`.
- Se verificó con medidas de test que Min < Max.

**Decisión tomada**
- Se creó `dim_date` con rango real entre `MIN(fact_orders[purchase_date])` y `MAX(...)`.
- Se marcó `dim_date` como tabla de fechas oficial en Power BI.

---

### Modelo estrella y control de relaciones automáticas

**Problema identificado**
Power BI detectó relaciones automáticas con tablas auxiliares (`agg_*`) que ya no eran necesarias porque la información estaba mergeada en `fact_orders`.

**Decisión tomada**
Se deshabilitó "Habilitar carga" en consultas auxiliares: `data_raw`, todas las `stg_*`, `agg_payments_order`, `agg_reviews_order`, `stg_category_translation`.

**Modelo final**
- `fact_orders`, `fact_sales`, `dim_customers`, `dim_products`, `dim_sellers`, `dim_date`, Medidas.

**Relaciones activas**

| Desde | Hacia |
|---|---|
| `fact_orders[customer_id]` | `dim_customers[customer_id]` |
| `fact_orders[purchase_date]` | `dim_date[Date]` |
| `fact_sales[customer_id]` | `dim_customers[customer_id]` |
| `fact_sales[product_id]` | `dim_products[product_id]` |
| `fact_sales[purchase_date]` | `dim_date[Date]` |
| `fact_sales[seller_id]` | `dim_sellers[seller_id]` |

**Reglas aplicadas**
- Cardinalidad: muchos a uno (* → 1) desde hechos hacia dimensiones.
- Dirección de filtro: única desde dimensiones hacia hechos.

---

## Visualizaciones

### Panel: Executive Overview

**Slicer YearMonth**
*Pregunta:* ¿Cómo cambian los KPIs si analizo un mes específico?

**KPI Cards**

| Card | Pregunta |
|---|---|
| Orders | ¿Cuántos pedidos se realizaron en el período? |
| GMV | ¿Cuánto se vendió (monto total pagado)? |
| Customers (unique) | ¿Cuántos clientes únicos compraron? |
| AOV | ¿Cuál fue el ticket promedio por pedido? |

**Gráfico de barras apiladas: Sales por categoría**
*Pregunta:* ¿Qué categorías explican el mayor volumen de ventas?
*Hipótesis:* Pocas categorías concentran la mayor parte (principio 80/20).

**Gráfico de líneas: GMV por año**
*Pregunta:* ¿La facturación está creciendo, estable o cayendo?

**Medidas adicionales**
- `Orders per Customer` = `DIVIDE([Orders], [Customers (unique)])`
- `GMV per Customer` = `DIVIDE([GMV], [Customers (unique)])`

---

### Panel: Growth Drivers & Mix

*Preguntas clave:*
- ¿El crecimiento de GMV está explicado por más clientes o por más pedidos?
- ¿Cambió el mix de categorías?
- ¿Qué categorías están empujando (o frenando) las ventas?

**Visualizaciones**
- Columnas agrupadas + líneas: GMV vs Orders (driver de volumen)
- Columnas 100% apiladas: mix de categorías en el tiempo
- Tooltip: GMV, Orders, Customers, AOV por mes

---

### Panel: Retención y Recompra (Cohortes)

**Columnas calculadas en fact_orders**
- `CohortMonth`: mes de primera compra del cliente
- `OrderMonth`: mes de cada pedido
- `MonthsSinceCohort`: meses transcurridos desde la primera compra

**Medidas**
- `Cohort Customers`: clientes activos por cohorte y mes
- `Cohort Size`: tamaño de la cohorte en mes 0
- `Retention %`: retención por `MonthsSinceCohort`
- `Returning Rate`: % de clientes que volvieron a comprar al menos una vez

**Matriz de cohortes — cómo leerla**

| Columna | Significado |
|---|---|
| M0 | Mes de primera compra (~100%) |
| M1 | % que vuelve al mes siguiente |
| M2 | % que vuelve al segundo mes |

*Permite identificar:*
- ¿Qué cohortes retienen mejor/peor?
- ¿La retención mejora o empeora con el tiempo?
- ¿En qué mes se pierde la mayoría de los clientes?

---

### Panel: Segmentación de Clientes

**Objetivo**
- ¿Quiénes son los clientes de mayor valor?
- ¿Qué proporción del GMV viene de clientes recurrentes vs nuevos?
- ¿Qué segmentos hay que reactivar vs premiar?

**Medidas**
- `Last Purchase Date`: última fecha de compra por cliente
- `Recency`: días desde última compra
- `Frequency`: pedidos por cliente
- `Monetary`: GMV por cliente

**Bandas de recencia**

| Banda | Rango | Interpretación |
|---|---|---|
| Active | 0–30 días | Clientes activos |
| At Risk | 31–90 días | En riesgo |
| Dormant | 91–180 días | Dormidos |
| Lost | 181+ días | Perdidos |

**Visualizaciones**
- Tabla: Top 20 clientes por valor
- Gráfico de barras: Customers por banda de recencia
- Gráfico de barras: GMV por banda de recencia

**Paleta de colores (inspirada en Olist)**
- Primary: `#0C29D0`
- Secondary: `#364EF7`
- Dark blue: `#0A1F9C`
- Light blues: `#7DA2FF`, `#ABC6FF`, `#E6EEFF`
- Accent: `#CE7B50`
