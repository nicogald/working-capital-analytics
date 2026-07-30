# Supply Chain Analytics — DistribuChile S.A.

## Contexto
DistribuChile S.A. es una empresa distribuidora con operaciones en todo el territorio nacional 
(Norte, Centro, Sur), con 3 categorías de producto (Electrónica, Alimentos, Textil). 
Este proyecto analiza el ciclo de capital de trabajo, la segmentación de inventario y el 
pronóstico de demanda durante el año 2023-2024, combinando KPIs financieros, análisis 
operativo de productos y modelos de forecasting.

---

## Problema
El CFO de DistribuChile S.A. necesitaba entender tres cosas:
1. El estado del capital de trabajo para identificar dónde se estaba perdiendo liquidez
2. Qué productos merecen mayor atención en la gestión de inventario y por qué
3. Cuánto reabastecer de los productos más críticos para evitar quiebres de stock en 2024

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

## ANÁLISIS 3 — Forecasting de Demanda

### Contexto
Los productos AY (P001, P002 y P004) presentaron quiebres de stock recurrentes durante 2023. 
Se aplicaron modelos de pronóstico de demanda para anticipar su reabastecimiento y evitar 
que el problema se repita en 2024.

### Metodología
- **Tratamiento de quiebres de stock**: los meses con inventario final en 0 (junio, septiembre 
  y diciembre) fueron calculados usando el promedio de los meses adyacentes sin quiebre, para 
  reconstruir una serie de demanda representativa en lugar de usar las ventas subregistradas.
- **Selección de modelo**: dado que el dataset cubre solo 12 meses (un ciclo anual), no hay 
  datos suficientes para Holt-Winters o SARIMA, que requieren al menos 2-3 ciclos completos. 
  Se seleccionó el método según las características de cada serie:
  - **P001** → Holt (tendencia descendente clara)
  - **P002** → SES (sin tendencia ni estacionalidad clara)
  - **P004** → Holt (tendencia descendente clara)
- **Validación**: cada modelo se entrenó con 9 meses (enero-septiembre) y se evaluó contra 
  los 3 meses restantes (octubre-diciembre) antes de reentrenar con los 12 meses completos 
  para el pronóstico final.

### Métricas de validación

| Producto | Modelo | MAPE | MAE | RMSE | Bias |
|---|---|---|---|---|---|
| P001 | Holt | 8.4% | 1.83 | 2.10 | 0.17 |
| P002 | SES | 5.6% | 1.17 | 1.32 | -0.50 |
| P004 | Holt | 4.6% | 1.67 | 1.73 | -1.00 |

Los tres modelos presentan MAPE inferior al 10%, considerado excelente en supply chain. 
El bias negativo en P002 y P004 indica una leve tendencia a subestimar la demanda, lo que 
se compensó agregando margen en la recomendación de pedido.

### Forecast enero-marzo 2024

| Período | P001 | P002 | P004 |
|---|---|---|---|
| Enero 2024 | 22 | 20 | 35 |
| Febrero 2024 | 22 | 20 | 35 |
| Marzo 2024 | 21 | 20 | 34 |

### Recomendación de reabastecimiento

| Producto | Stock actual (dic 2023) | Demanda pronosticada (Q1 2024) | Unidades a pedir |
|---|---|---|---|
| P001 Laptop ProBook | 0 | 65 | **65** |
| P002 Monitor 24" | 0 | 60 | **60** |
| P004 Mouse Optico | 0 | 104 | **104** |

Los tres productos AY terminaron 2023 con inventario en 0, confirmando el problema de 
abastecimiento identificado en el análisis de working capital. Sin una acción correctiva 
inmediata, los quiebres de stock continuarían en el primer trimestre de 2024.

---

## Conclusión 

El DIO artificialmente bajo de 6 días no refleja una rotación eficiente sino quiebres de stock recurrentes en los productos de mayor valor de la categoría Electrónica. La segmentación ABC-XYZ confirma que estos productos (AY) son críticos para el negocio y requieren una política de inventario más robusta. El forecast de demanda cuantifica el problema: sin intervención, P001, P002 y P004 mantendrían inventario en 0 durante todo el primer trimestre de 2024. La recomendación de pedido (65, 60 y 104 unidades respectivamente) le da al equipo de compras un número concreto para negociar con proveedores, en lugar de reabastecer de forma reactiva como ha ocurrido hasta ahora. Cubrir esta demanda elevaría el DIO a un valor más real, reduciría las ventas perdidas por falta de stock y en consecuencia mejoraría el DSO y el CCC.

---

## Recomendaciones

1. **Reducir DSO** — implementar descuentos por pronto pago para clientes en la zona Sur y alertas automáticas para facturas vencidas.

2. **Ejecutar el pedido de reabastecimiento** — solicitar 65 unidades de P001, 60 de P002 y 104 de P004 antes de enero 2024 para cubrir la demanda proyectada del Q1 y eliminar los quiebres de stock recurrentes.

3. **Aumentar DPO** — negociar plazos de pago más largos con proveedores clave, especialmente con aquellos que actualmente se pagan en menos de 30 días, para liberar capital de trabajo.

4. **Automatizar gestión de productos CX** — P006 y P007 pueden gestionarse con reorden automático, liberando tiempo del equipo para enfocarse en los productos AY.

5. **Ampliar el horizonte de datos** — con más de 12 meses de historia se podrían aplicar Holt-Winters o SARIMA para capturar estacionalidad y mejorar la precisión del forecast.

---

## Herramientas utilizadas
- **Python** — pandas, matplotlib, statsmodels (limpieza de datos, cálculo de KPIs y forecasting)
- **Google Colab** — entorno de desarrollo
- **Power BI** — dashboard ejecutivo de 5 páginas

---

## Dashboard
<img width="941" height="540" alt="Dashboard kpis DistribuChile" src="https://github.com/user-attachments/assets/968f4a29-b893-4327-bfdf-9bb8c687276e" />
<br><br>
<img width="955" height="535" alt="kpis analisis x proveedores" src="https://github.com/user-attachments/assets/03b90950-2f38-4d83-aa72-edb298c6067e" />
<br><br>
<img width="952" height="535" alt="Analisis inventario y cobro proyecto kpis" src="https://github.com/user-attachments/assets/e871a688-49b7-46cd-9d70-12c8b5629663" />
<br><br>
<img width="1863" height="1054" alt="Segmentacion ABC-XYZ" src="https://github.com/user-attachments/assets/f2f16bf0-09fa-47f6-8388-48e299d0cff8" />
<br><br>
<img width="1411" height="764" alt="image" src="https://github.com/user-attachments/assets/9c3e7e97-63f0-4efe-9581-fc86e73b3205" />

---

## Estructura del repositorio
```text
├── datos_crudos/
│   ├── ventas_cuentas_cobrar.csv
│   ├── inventario.csv
│   ├── cuentas_pagar.csv
│   └── ventas_por_producto.csv
├── datos_limpios/
│   ├── ventas_limpio.csv
│   ├── inventario_limpio.csv
│   ├── compras_limpio.csv
│   ├── dpo_por_proveedor.csv
│   ├── dso_regional.csv
│   ├── dso_mensual.csv
│   ├── dio_por_categoria.csv
│   ├── kpis_claves.csv
│   ├── tabla_abc.csv
│   ├── tabla_xyz.csv
│   ├── tabla_combinada.csv
│   ├── tabla_combinada_valores.csv
│   ├── resumen_metricas_prediccion.csv
│   ├── resumen_prediccion_proyecto.csv
│   └── tabla_combinada_inventario_final_pronostico.csv
├── notebooks/
│   ├── 01_notebook_analisis_capital.ipynb
├── dashboard/
│   └── Proyecto_kpis.pbix
└── README.md
```
