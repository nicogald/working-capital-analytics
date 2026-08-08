# Supply Chain Analytics — DistribuChile S.A.

## Contexto
DistribuChile S.A. es una empresa distribuidora con operaciones en todo el territorio nacional 
(Norte, Centro, Sur), con 3 categorías de producto (Electrónica, Alimentos, Textil). 
Este proyecto analiza el ciclo de capital de trabajo, la segmentación de inventario, el 
pronóstico de demanda y la política de reabastecimiento durante el año 2023-2024, combinando 
KPIs financieros, análisis operativo de productos, modelos de forecasting y simulación 
Monte Carlo.

---

## Problema
El CFO de DistribuChile S.A. necesitaba entender cuatro cosas:
1. El estado del capital de trabajo para identificar dónde se estaba perdiendo liquidez
2. Qué productos merecen mayor atención en la gestión de inventario y por qué
3. Cuánto reabastecer de los productos más críticos para evitar quiebres de stock en 2024
4. Con qué cantidad de pedido (Q) y punto de reorden (r) sostener el inventario en el tiempo, 
   minimizando el costo total considerando la incertidumbre real de la demanda

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

## ANÁLISIS 4 — Política de Inventario (Q,r) con Simulación Monte Carlo

### Contexto
Con el forecast de demanda del Análisis 3 y su margen de error (RMSE), se calculó la política 
de inventario óptima (cantidad a pedir Q y punto de reorden r) para los productos AY, 
considerando el costo de quiebre de stock que las fórmulas clásicas de EOQ/ROP no contemplan.

### Metodología
- **Granularidad mensual**: dado que el forecast y RMSE del Análisis 3 están en base mensual, 
  la simulación avanza mes a mes (no día a día) para mantener consistencia de unidades.
- **Distribución de demanda**: se modeló como Normal, usando el forecast como media y el RMSE 
  como desviación estándar (válido dado que el Bias de los 3 modelos es cercano a cero). 
  Limitación reconocida: con solo 12 observaciones históricas no fue posible validar 
  estadísticamente este supuesto.
- **Búsqueda grueso → fino**: los rangos de Q y r se derivaron de las fórmulas clásicas de 
  EOQ y ROP (no arbitrariamente), usándolas como centro de un rango amplio (fase gruesa) y 
  luego refinando alrededor del mejor resultado (fase fina).
- **Common Random Numbers (CRN)**: todas las combinaciones de (Q,r) se evaluaron bajo la 
  misma secuencia de demanda simulada, para comparar limpio sin que el azar favorezca a una 
  política sobre otra.
- **N óptimo de réplicas**: se calibró el número de réplicas necesario mediante una corrida 
  piloto (200 réplicas) y la fórmula N = (Z·σ/E)², usando un margen de error del 1% sobre el 
  costo esperado. Esto evitó usar un número arbitrario de réplicas y redujo el costo 
  computacional sin perder precisión.

### Supuestos operativos
| Parámetro | Valor | Fuente |
|---|---|---|
| Lead time | 15 días (0.5 meses) | Supuesto documentado |
| Tasa de costo de capital/almacenaje | 20% anual | Supuesto documentado |
| Costo de ordenar | $50.000 CLP | Supuesto documentado |
| Costo de quiebre | 30% del costo unitario | Supuesto documentado |

### Resultados — Política óptima por producto

| Producto | N réplicas | Q óptimo | r óptimo | Costo esperado anual | Nivel de servicio |
|---|---|---|---|---|---|
| P001 Laptop ProBook | 59 | 47 | 42 | $7.774.628 | 83.2% |
| P002 Monitor 24" | 54 | 44 | 37 | $3.085.243 | 83.2% |
| P004 Mouse Optico | 30 | 106 | 48 | $707.445 | 83.1% |

### Hallazgo principal
Ninguna combinación de (Q,r) evaluada alcanzó el 95% de nivel de servicio objetivo, partiendo 
del inventario real en 0 al cierre de 2023. Esto confirma matemáticamente que la política de 
mantenimiento (Q,r) por sí sola no puede resolver un déficit de stock preexistente: se 
requiere primero el pedido de reabastecimiento inicial (65, 60 y 104 unidades, calculado en 
el Análisis 3) para sacar el sistema de déficit, y recién después esta política sostiene el 
inventario establemente en torno al 83% de servicio con el costo mínimo posible.

### Trabajo futuro
Con más historia de ventas se podría: (1) validar estadísticamente la distribución de demanda 
con un test de bondad de ajuste, (2) contrastar el supuesto Normal contra un bootstrap de 
residuos históricos, (3) obtener lead times y costos reales de la empresa en vez de los 
supuestos documentados aquí, y (4) extender la política de inventario al resto del 
portafolio: ROP clásico (revisión continua) para productos AX, y política de revisión 
periódica (R,S) para productos BX, BY y CX.

---

## Conclusión 

El DIO artificialmente bajo de 6 días no refleja una rotación eficiente sino quiebres de stock recurrentes en los productos de mayor valor de la categoría Electrónica. La segmentación ABC-XYZ confirma que estos productos (AY) son críticos para el negocio y requieren una política de inventario más robusta. El forecast de demanda cuantifica el problema: sin intervención, P001, P002 y P004 mantendrían inventario en 0 durante todo el primer trimestre de 2024. La simulación Monte Carlo va un paso más allá: confirma que ni siquiera una política de reorden bien calibrada puede sostener el servicio sin antes resolver el déficit inicial, y entrega la cantidad exacta (Q) y el punto de reorden (r) que minimizan el costo total una vez cubierto ese déficit. En conjunto, estos cuatro análisis le dan al equipo de compras un plan concreto: cuánto pedir ahora (65, 60 y 104 unidades), y cómo gestionar el inventario de ahí en adelante (Q y r óptimos por producto) para mantener el costo mínimo posible dado el nivel de servicio alcanzable.

---

## Recomendaciones

1. **Reducir DSO** — implementar descuentos por pronto pago para clientes en la zona Sur y alertas automáticas para facturas vencidas.

2. **Ejecutar el pedido de reabastecimiento inicial** — solicitar 65 unidades de P001, 60 de P002 y 104 de P004 antes de enero 2024 para cubrir la demanda proyectada del Q1 y sacar el sistema del déficit de stock.

3. **Adoptar la política (Q,r) calibrada** — una vez cubierto el déficit inicial, ordenar en lotes de 47 (P001), 44 (P002) y 106 (P004) unidades cada vez que el inventario llegue a 42, 37 y 48 unidades respectivamente.

4. **Aumentar DPO** — negociar plazos de pago más largos con proveedores clave, especialmente con aquellos que actualmente se pagan en menos de 30 días, para liberar capital de trabajo.

5. **Automatizar gestión de productos CX** — P006 y P007 pueden gestionarse con reorden automático o revisión periódica, liberando tiempo del equipo para enfocarse en los productos AY.

6. **Ampliar el horizonte de datos** — con más de 12 meses de historia se podrían aplicar Holt-Winters o SARIMA para capturar estacionalidad, validar estadísticamente la distribución de demanda, y extender la política de inventario al resto del portafolio.

---

## Herramientas utilizadas
- **Python** — pandas, numpy, matplotlib, statsmodels, scipy (limpieza de datos, cálculo de KPIs, forecasting y simulación Monte Carlo)
- **Google Colab** — entorno de desarrollo
- **Power BI** — dashboard ejecutivo de 6 páginas

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
<img width="1411" height="764" alt="Forecasting de demanda" src="https://github.com/user-attachments/assets/9c3e7e97-63f0-4efe-9581-fc86e73b3205" />
<br><br>
<img width="1357" height="762" alt="Politica de inventario Q,r Monte Carlo" src="PEGA_AQUI_EL_LINK_DE_TU_CAPTURA" />

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
│   ├── tabla_combinada_inventario_final_pronostico.csv
│   ├── politica_inventario_resumen.csv
│   └── combinaciones_todos_productos.csv
├── notebooks/
│   ├── 01_notebook_analisis_capital.ipynb
│   └── 02_simulacion_montecarlo_inventario.py
├── dashboard/
│   └── Proyecto_kpis.pbix
└── README.md
```
