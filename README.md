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
| inventario.csv | Movimientos mensuales de stock | 130 |
| cuentas_pagar.csv | Órdenes de compra a proveedores | 128 |

---

## Proceso de limpieza
Cada dataset contenía errores intencionales que fueron identificados y resueltos:

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

**DSO — 73 días**
La empresa tarda en promedio 73 días en cobrar a sus clientes. 
El mes con mayor DSO fue agosto y diciembre, y la región con 
mayor DSO fue [región con mayor DSO]. Esto indica oportunidades 
de mejora en la gestión de cobranza.

**DIO — 6 días**
El inventario rota cada 6 días en promedio. Este valor está 
influenciado por quiebres de stock recurrentes en varios productos 
que reducen artificialmente el inventario promedio. En un escenario 
real esto indicaría un problema de abastecimiento más que una 
rotación eficiente.

**DPO — 18 días**
La empresa paga a sus proveedores en promedio en 18 días. Este valor 
está fuertemente influenciado por PR006, proveedor con el mayor volumen 
de compras del año ($1.064.199.999) y con casi todo su saldo pagado. 
Excluyendo PR006, el DPO sube significativamente, revelando que con el 
resto de proveedores la empresa paga muy rápido perdiendo liquidez 
innecesariamente.

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

2. **Mejorar DIO** — revisar la política de reorden para eliminar quiebres 
   de stock recurrentes, especialmente en categoría Electrónica.

3. **Aumentar DPO** — negociar plazos de pago más largos con proveedores 
   clave, especialmente con aquellos que actualmente se pagan en menos de 
   30 días, para liberar capital de trabajo.

---

## Herramientas utilizadas
- **Python** — pandas (limpieza de datos y cálculo de KPIs)
- **Google Colab** — entorno de desarrollo
- **Power BI** — dashboard ejecutivo de 3 páginas

---

## Estructura del repositorio
