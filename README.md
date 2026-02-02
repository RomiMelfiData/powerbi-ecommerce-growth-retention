# E-commerce Growth & Retention (Power BI) | Olist Dataset

## Resumen
Proyecto de portfolio en **Power BI** para analizar un e-commerce de punta a punta: **crecimiento**, **retención (cohortes)**, **performance logística (SLA)** y **satisfacción del cliente (reviews)**.  
Incluye **modelado en esquema estrella**, **definición de KPIs**, **transformaciones en SQL** y **controles de calidad de datos**.

### Objetivo
Construir un dashboard ejecutivo y operativo que responda preguntas clave:
- ¿Cómo evolucionan pedidos, ventas (GMV) y ticket promedio (AOV)?
- ¿Qué cohortes retienen mejor/peor y cómo cambia la recompra?
- ¿Dónde se rompe la experiencia logística (demoras, entregas fuera de término)?
- ¿Qué categorías/sellers impulsan crecimiento y cuáles generan fricción (cancelaciones, malas reviews)?

### Dataset
**Olist Brazilian E-Commerce Public Dataset (Kaggle)**.  
> Nota: por buenas prácticas, los datos **no se suben al repo**.  
Descarga manual desde Kaggle y colocación en `data_raw/`.

### Estructura del repositorio
- `powerbi/` → archivo del reporte (.pbix o template .pbit)
- `sql/` → queries de staging/marts + checks de calidad
- `docs/` → brief, diccionario de métricas y documentación del modelo
- `images/` → capturas del dashboard para el README
- `data_raw/` → datos originales (ignorado por git)
- `data_processed/` → datos procesados (ignorado por git)

### Cómo reproducir (pasos)
1) Descargar el dataset Olist desde Kaggle y descomprimirlo en `data_raw/`.
2) Abrir el archivo de Power BI en `powerbi/`.
3) Actualizar rutas/orígenes si corresponde y refrescar el modelo.

### KPIs (versión inicial)
- **Orders**: cantidad de pedidos
- **Customers**: clientes únicos
- **GMV**: suma de `payment_value`
- **AOV**: GMV / Orders
- **Repeat rate**: % de clientes con 2+ pedidos
- **On-time delivery rate**: % entregas en o antes de la fecha estimada
- **Avg delivery delay (days)**: (delivered - estimated) en días

---

## EN — Overview
Portfolio **Power BI** project to analyze an e-commerce business end-to-end: **growth**, **cohort retention**, **logistics performance (SLA)** and **customer satisfaction (reviews)**.  
It includes **star schema modeling**, **KPI definitions**, **SQL transformations**, and **data quality checks**.

### Goal
Build an executive + operational dashboard to answer:
- How do orders, GMV and AOV evolve over time?
- Which cohorts retain better/worse and how repeat purchase changes?
- Where does the logistics experience break (delays, late deliveries)?
- Which categories/sellers drive growth and which create friction (cancellations, bad reviews)?

### Dataset
**Olist Brazilian E-Commerce Public Dataset (Kaggle)**.  
> Note: raw data is **not committed** to the repository.  
Manual download and placement under `data_raw/`.

### Repository structure
- `powerbi/` → report file (.pbix or template .pbit)
- `sql/` → staging/marts queries + quality checks
- `docs/` → brief, metric dictionary, model documentation
- `images/` → dashboard screenshots for the README
- `data_raw/` → raw files (git ignored)
- `data_processed/` → processed files (git ignored)

### Repro steps
1) Download the Olist dataset from Kaggle and unzip it into `data_raw/`.
2) Open the Power BI file under `powerbi/`.
3) Update paths/sources if needed and refresh the model.

---

## Capturas / Screenshots
(Agregar imágenes en `images/` y linkearlas acá cuando el dashboard esté listo.)

## Próximos pasos
- Definir el modelo estrella (fact + dims) y el grano de cada tabla.
- Implementar cohortes de retención y métricas logísticas.
- Documentar el diccionario de métricas y reglas de calidad.
