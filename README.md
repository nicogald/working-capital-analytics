# Supply Chain Analytics — DistribuChile S.A.

## Contexto
DistribuChile S.A. es una empresa distribuidora con operaciones en todo el territorio nacional 
(Norte, Centro, Sur), con 3 categorías de producto (Electrónica, Alimentos, Textil). 
Este proyecto analiza el ciclo de capital de trabajo y la segmentación de inventario 
durante el año 2023, combinando KPIs financieros con análisis operativo de productos.

---

## Problema
El CFO de DistribuChile S.A. necesitaba entender dos cosas:
1. El estado del capital de trabajo para identificar dónde se estaba perdiendo liquidez
2. Qué productos merecen mayor atención en la gestión de inventario y por qué

---

## Datos
El análisis se construyó sobre 4 datasets:

| Archivo | Descripción | Registros |
|---|---|---|
| ventas_cuentas_cobrar.csv | Facturas emitidas a clientes | 150 |
| inventario.csv | Movimientos mensuales de stock por producto | 126 |
| cuentas_pagar.csv | Órdenes de compra a proveedores | 128 |
| ventas_por_producto.csv | Ventas mensuales por producto | 120 |

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

## ANÁLISIS 1 — Working Capital

### KPIs calculados

| KPI | Fórmula | Resultado |
|---|---|---|
| DSO | (CxC pendiente / Ventas totales) × 365 | 73 días |
| DIO | (Inventario promedio / Costo de ventas) × 365 | 6 días |
| DPO | (CxP pendiente / Costo de compras) × 365 | 18 días |
| CCC | DSO + DIO − DPO | 61 días |

### Hallazgos

**DSO — 73 días**
La empresa tarda en promedio 73 días en cobrar a sus clientes. La zona Sur presenta el mayor DSO del país, lo que indica oportunidades de mejora en la gestión de cobranza en esa región.

**DIO — 6 días**
El DIO de 6 días es artificialmente bajo. Se identificaron 10 registros con inventario final igual a 0, todos en productos de Electrónica (P001, P002, P003 y P004), concentrados en la segunda mitad de 2023. Estos quiebres de stock reducen el inventario promedio y distorsionan el KPI hacia abajo, ocultando un problema real de abastecimiento.

**DPO — 18 días**
El DPO global de 18 días está fuertemente influenciado por PR006, proveedor con el mayor volumen de compras del año ($1.064.199.999) y con casi todo su saldo pagado. Al excluir PR006, el DPO sube significativamente, revelando que con el resto de proveedores la empresa tiene mayor deuda pendiente, lo que indica que PR006 estaba bajando artificialmente el promedio global.

**CCC — 61 días**
La empresa tiene 61 días de capital propio comprometido en el ciclo operativo. Desde que paga a sus proveedores hasta que cobra a sus clientes pasan 61 días que debe financiar con recursos propios o crédito bancario.

---

## ANÁLISIS 2 — Segmentación ABC-XYZ

### Metodología
- **ABC** → clasificación por valor de ventas anuales (cortes en 80% y 95%)
- **XYZ** → clasificación por variabilidad de demanda usando coeficiente de variación (CV)
  - X: CV < 20% → demanda estable
  - Y: CV entre 20% y 50% → demanda variable
  - Z: CV > 50% → demanda impredecible

### Resultados

| Producto | Categoría | Observación |
|---|---|---|
| P003 Teclado Inalambrico | AX | Alto valor, demanda estable |
| P008 Polera Algodon M | AX | Alto valor, demanda estable |
| P001 Laptop ProBook | AY | Alto valor, demanda variable + quiebres de stock |
| P002 Monitor 24" | AY | Alto valor, demanda variable + quiebres de stock |
| P004 Mouse Optico | AY | Alto valor, demanda variable + quiebres de stock |
| P009 Pantalon Jean 32 | BY | Valor medio, demanda variable |
| P005 Arroz Grano Largo | BX | Valor medio, demanda estable |
| P010 Zapatilla Running | BX | Valor medio, demanda estable |
| P006 Aceite Vegetal 1L | CX | Bajo valor, demanda estable |
| P007 Harina Sin Polvos | CX | Bajo valor, demanda estable |

### Hallazgos

Los productos clasificados como **AY** (P001, P002 y P004) son los más críticos del portafolio. Son de alto valor económico y presentan demanda variable, lo que dificulta la planificación del reabastecimiento. Adicionalmente, estos mismos productos concentraron todos los quiebres de stock del año, lo que sugiere que su variabilidad no es solo natural sino que está siendo amplificada por una política de reabastecimiento insuficiente.

Los productos **AX** (P003 y P008) son los más fáciles de gestionar: alto valor y demanda predecible. Son candidatos a gestión con stock ajustado sin necesidad de grandes colchones de seguridad.

Los productos **CX** (P006 y P007) de bajo valor y demanda estable pueden gestionarse con reorden automático sin atención manual.

---

## Conclusión integrada

El DIO artificialmente bajo de 6 días no refleja una rotación eficiente sino quiebres de stock recurrentes en los productos de mayor valor de la categoría Electrónica. La segmentación ABC-XYZ confirma que estos productos (AY) son críticos para el negocio y requieren una política de inventario más robusta. Mejorar el abastecimiento de los productos AY elevaría el DIO a un valor más real pero también reduciría las ventas perdidas por falta de stock, mejorando el DSO y en consecuencia el CCC.

---

## Recomendaciones

1. **Reducir DSO** — implementar descuentos por pronto pago para clientes en la zona Sur y alertas automáticas para facturas vencidas.

2. **Corregir política de reabastecimiento en productos AY** — establecer stock de seguridad y puntos de reorden específicos para P001, P002 y P004 para eliminar los quiebres de stock recurrentes en la segunda mitad del año.

3. **Aumentar DPO** — negociar plazos de pago más largos con proveedores clave, especialmente con aquellos que actualmente se pagan en menos de 30 días, para liberar capital de trabajo.

4. **Automatizar gestión de productos CX** — P006 y P007 pueden gestionarse con reorden automático, liberando tiempo del equipo para enfocarse en los productos AY.

---

## Herramientas utilizadas
- **Python** — pandas (limpieza de datos y cálculo de KPIs)
- **Google Colab** — entorno de desarrollo
- **Power BI** — dashboard ejecutivo de 4 páginas

---

## Dashboard
<img width="941" height="540" alt="Dashboard kpis DistribuChile" src="https://github.com/user-attachments/assets/968f4a29-b893-4327-bfdf-9bb8c687276e" />
<br><br>
<img width="955" height="535" alt="kpis analisis x proveedores" src="https://github.com/user-attachments/assets/03b90950-2f38-4d83-aa72-edb298c6067e" />
<br><br>
<img width="952" height="535" alt="Analisis inventario y cobro proyecto kpis" src="https://github.com/user-attachments/assets/e871a688-49b7-46cd-9d70-12c8b5629663" />
<br><br>
<img width="1863" height="1054" alt="Captura de pantalla 2026-05-25 220735" src="https://github.com/user-attachments/assets/f2f16bf0-09fa-47f6-8388-48e299d0cff8" />

---

## Estructura del repositorio
```text
├── datos_crudos/
│   ├── ventas_cuentas_cobrar.csv
│   ├── inventario.csv
│   └── cuentas_pagar.csv
│   └── ventas_por_producto(2).csv
├── datos_limpios/
│   ├── ventas_limpio.csv
│   ├── inventario_limpio.csv
│   ├── compras_limpio.csv
│   ├── ventas_por_producto.csv
│   ├── dpo_por_proveedor.csv
│   ├── dso_regional.csv
│   ├── dso_mensual.csv
│   ├── dio_por_categoria.csv
│   ├── kpis_claves.csv
│   ├── tabla_abc.csv
│   ├── tabla_xyz.csv
│   ├── tabla_combinada.csv
│   └── tabla_comibnada_valores.csv
├── notebooks/
 ├── 01_Notebook_analisis_capital.ipynb
│   
├── dashboard/
│   └── Proyecto_kpis.pbix
└── README.md
```
