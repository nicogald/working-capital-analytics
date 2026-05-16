# Working Capital Analytics — DistribuChile S.A.

## Contexto
DistribuChile S.A. es una empresa distribuidora con operaciones en todo el territorio nacional 
(Norte, Centro, Sur), cuenta con 3 categorías de producto (Electrónica, Alimentos, Textil). 
Este proyecto analiza el ciclo de capital de trabajo de la empresa durante el año 2023, 
calculando los KPIs financieros DSO, DIO, DPO y CCC a partir de datos operacionales reales.

---

## Problema
El CFO de DistribuChile S.A. necesitaba entender el estado del capital de trabajo 
de la empresa para identificar en qué parte del ciclo operativo se estaba perdiendo 
liquidez y tomar decisiones informadas sobre cobro, inventario y gestión de proveedores.

---

## Datos
El análisis se construyó sobre 3 datasets con datos sucios que requirieron limpieza 
exhaustiva antes de cualquier cálculo:

| Archivo | Descripción | Registros |
|---|---|---|
| ventas_cuentas_cobrar.csv | Facturas emitidas a clientes | 150 |
| inventario.csv | Movimientos mensuales de stock por producto | 126 |
| cuentas_pagar.csv | Órdenes de compra a proveedores | 128 |

---

## Proceso de limpieza
Cada dataset contenía errores que fueron identificados y resueltos:

**ventas_cuentas_cobrar.csv**
- Facturas sin ID y con ID duplicado (F-0149 con dos clientes distintos)
- Fechas de venta con formato inválido y fechas de pago anteriores a la venta
- Montos negativos y outlier extremo de $999.999.999
- Cliente C002 con dos nombres distintos en el sistema

**inventario.csv**
- Producto con inventario inicial negativo (-50 unidades)
- Registros sin fecha, sin nombre de producto y sin región de bodega
- Duplicados en producto P001 y P005
- Inconsistencias en la identidad contable del inventario

**cuentas_pagar.csv**
- Órdenes con ID inválido y sin ID
- Fecha de pago acordado anterior a fecha de compra
- Monto negativo corregido a valor absoluto
- Error en año bisiesto 2024 detectado y corregido en OC-111

---

## KPIs calculados

| KPI | Fórmula | Resultado |
|---|---|---|
| DSO | (CxC pendiente / Ventas totales) × 365 | 73 días |
| DIO | (Inventario promedio / Costo de ventas) × 365 | 6 días |
| DPO | (CxP pendiente / Costo de compras) × 365 | 18 días |
| CCC | DSO + DIO − DPO | 61 días |

---

## Hallazgos principales

**DSO GLOBAL — 73 días**
La empresa tarda en promedio 73 días en cobrar a sus clientes. 
El mes con mayor DSO fue agosto y diciembre, y la zona del pais con 
mayor DSO fue zona sur. Esto indica oportunidades 
de mejora en la gestión de cobranza en esa zona en especifico.

**DIO GLOBAL — 6 días**
El inventario rota cada 6 días en promedio. Este valor está 
influenciado por quiebres de stock recurrentes en varios productos que se pueden evidenciar al ver las tablas en donde varios productos tienen inventario_final igual a cero,
esto reduce artificialmente el inventario promedio. Esto podría indicar 
 un problema de abastecimiento más que una 
rotación eficiente.

**DPO GLOBAL — 18 días**
La empresa paga a sus proveedores en promedio en 18 días. Este valor está fuertemente influenciado por PR006, proveedor con el mayor volumen de compras del año y con gran parte de sus pagos ya realizados. Al excluir PR006, el DPO aumenta significativamente, revelando que con el resto de proveedores la empresa demora más en efectuar los pagos.

**CCC — 61 días**
La empresa tiene 61 días de capital propio comprometido en el ciclo 
operativo. Esto significa que desde que paga a sus proveedores hasta 
que cobra a sus clientes pasan 61 días que debe financiar con recursos 
propios o crédito bancario.

---

## Recomendaciones

1. **Reducir DSO** — implementar descuentos por pronto pago para clientes 
   en la región con mayor DSO y establecer alertas automáticas para facturas 
   vencidas.

2. **Mejorar DIO** — revisar política de inventario y niveles de reorden para optimizar rotación y reducir exceso de inventario, especialmente en Electrónica.

3. **Aumentar DPO** — negociar plazos de pago más largos con proveedores 
   clave, especialmente con aquellos que actualmente se pagan en menos de 
   30 días, para liberar capital de trabajo.

---

## Herramientas utilizadas
- **Python** — pandas (limpieza de datos y cálculo de KPIs)
- **Google Colab** — entorno de desarrollo
- **Power BI** — dashboard ejecutivo de 3 páginas

---
## Dashboard
<img width="941" height="540" alt="Dashboard kpis DistribuChile" src="https://github.com/user-attachments/assets/968f4a29-b893-4327-bfdf-9bb8c687276e" />
---
<img width="955" height="535" alt="kpis analisis x proveedores" src="https://github.com/user-attachments/assets/03b90950-2f38-4d83-aa72-edb298c6067e" />
---
<img width="952" height="535" alt="Analisis inventario y cobro proyecto kpis" src="https://github.com/user-attachments/assets/e871a688-49b7-46cd-9d70-12c8b5629663" />


---
## Estructura del repositorio
