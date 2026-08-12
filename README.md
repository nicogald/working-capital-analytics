# Supply Chain Analytics — DistribuChile S.A.

## Contexto

DistribuChile S.A. es una empresa distribuidora con operaciones a nivel nacional, divididas en las zonas Norte, Centro y Sur, y cuenta con tres categorías de productos: Electrónica, Alimentos y Textil.

El objetivo de este proyecto es analizar cómo se está comportando la operación desde cuatro perspectivas relacionadas entre sí: capital de trabajo, gestión de inventario, pronóstico de demanda y política de reabastecimiento.

Para esto, se combinaron KPIs financieros y operativos, segmentación de productos, modelos de forecasting y una simulación Monte Carlo para evaluar distintas políticas de inventario.

---

## Problema

El CFO de DistribuChile S.A. necesitaba responder cuatro preguntas principales:

1. ¿Cuál es el estado actual del capital de trabajo y dónde se está concentrando la liquidez?
2. ¿Qué productos requieren mayor atención en la gestión de inventario y por qué?
3. ¿Cuánto se debería reabastecer de los productos más críticos para evitar nuevos quiebres de stock en 2024?
4. ¿Qué cantidad de pedido (Q) y punto de reorden (r) permiten mantener el inventario en el tiempo, minimizando el costo total bajo incertidumbre de demanda?

---

## Datos

El análisis se construyó a partir de 4 datasets:

| Archivo                   | Descripción                                 | Registros |
| ------------------------- | ------------------------------------------- | --------- |
| ventas_cuentas_cobrar.csv | Facturas emitidas a clientes                | 150       |
| inventario.csv            | Movimientos mensuales de stock por producto | 126       |
| cuentas_pagar.csv         | Órdenes de compra a proveedores             | 128       |
| ventas_por_producto.csv   | Ventas mensuales por producto               | 120       |

---

## Proceso de limpieza

Antes de realizar los análisis, se revisó cada dataset para identificar problemas de calidad de datos y corregirlos. Entre los principales hallazgos estuvieron:

**ventas_cuentas_cobrar.csv**

* Facturas sin ID y un ID duplicado (F-0149 asociado a dos clientes distintos).
* Fechas de venta con formato inválido y fechas de pago anteriores a la fecha de venta.
* Montos negativos y un outlier extremo de $999.999.999.
* El cliente C002 aparecía registrado con dos nombres distintos.

**inventario.csv**

* Un producto presentaba un inventario inicial negativo (-50 unidades).
* Registros sin fecha, nombre de producto o región de bodega.
* Registros duplicados para los productos P001 y P005.
* Inconsistencias en la identidad contable del inventario.

**cuentas_pagar.csv**

* Órdenes con ID inválido o sin ID.
* Fechas de pago acordado anteriores a la fecha de compra.
* Un monto negativo que fue corregido a su valor absoluto.
* Un error asociado al año bisiesto 2024 detectado y corregido en la OC-111.

---

## ANÁLISIS 1 — Working Capital

### KPIs calculados

| KPI | Fórmula                                       | Resultado |
| --- | --------------------------------------------- | --------- |
| DSO | (CxC pendiente / Ventas totales) × 365        | 73 días   |
| DIO | (Inventario promedio / Costo de ventas) × 365 | 6 días    |
| DPO | (CxP pendiente / Costo de compras) × 365      | 18 días   |
| CCC | DSO + DIO − DPO                               | 61 días   |

### Hallazgos

**DSO — 73 días**

En promedio, la empresa tarda 73 días en cobrar sus ventas. La zona Sur presenta el DSO más alto del país, por lo que existe una oportunidad de mejora en la gestión de cobranza de esa región.

**DIO — 6 días**

A primera vista, un DIO de 6 días podría parecer una buena señal de eficiencia. Sin embargo, al revisar los datos se encontró que este valor está artificialmente bajo.

Se identificaron 10 registros con inventario final igual a 0, todos correspondientes a productos de Electrónica (P001, P002, P003 y P004) y concentrados durante la segunda mitad de 2023.

Estos quiebres de stock reducen el inventario promedio y, por lo tanto, hacen que el DIO parezca más bajo de lo que realmente sería en una operación sin quiebres. En este caso, el KPI está ocultando un problema de abastecimiento.

**DPO — 18 días**

El DPO global de 18 días está fuertemente influenciado por el proveedor PR006. Este proveedor concentró el mayor volumen de compras del año ($1.064.199.999) y tenía prácticamente todo su saldo pagado.

Al excluir PR006 del análisis, el DPO aumenta de forma significativa. Esto muestra que el promedio global estaba siendo reducido por el comportamiento de este proveedor y que, con el resto de los proveedores, la empresa mantiene una mayor cantidad de deuda pendiente.

**CCC — 61 días**

El ciclo de conversión de efectivo es de 61 días. En términos simples, desde que la empresa paga a sus proveedores hasta que logra recuperar ese dinero mediante el cobro a sus clientes transcurren 61 días.

Durante ese período, la operación debe financiarse con recursos propios o mediante crédito.

---

## ANÁLISIS 2 — Segmentación ABC-XYZ

### Metodología

* **ABC** → clasificación según el valor de las ventas anuales, utilizando cortes en 80% y 95%.
* **XYZ** → clasificación según la variabilidad de la demanda mediante el coeficiente de variación (CV).

  * X: CV < 20% → demanda estable.
  * Y: CV entre 20% y 50% → demanda variable.
  * Z: CV > 50% → demanda impredecible.

### Resultados

| Producto                 | Categoría | Observación                                      |
| ------------------------ | --------- | ------------------------------------------------ |
| P003 Teclado Inalambrico | AX        | Alto valor, demanda estable                      |
| P008 Polera Algodon M    | AX        | Alto valor, demanda estable                      |
| P001 Laptop ProBook      | AY        | Alto valor, demanda variable + quiebres de stock |
| P002 Monitor 24"         | AY        | Alto valor, demanda variable + quiebres de stock |
| P004 Mouse Optico        | AY        | Alto valor, demanda variable + quiebres de stock |
| P009 Pantalon Jean 32    | BY        | Valor medio, demanda variable                    |
| P005 Arroz Grano Largo   | BX        | Valor medio, demanda estable                     |
| P010 Zapatilla Running   | BX        | Valor medio, demanda estable                     |
| P006 Aceite Vegetal 1L   | CX        | Bajo valor, demanda estable                      |
| P007 Harina Sin Polvos   | CX        | Bajo valor, demanda estable                      |

### Hallazgos

Los productos clasificados como **AY** (P001, P002 y P004) son los más críticos del portafolio. Combinan un alto valor económico con una demanda variable, por lo que requieren una planificación de inventario más cuidadosa.

Además, estos tres productos concentran todos los quiebres de stock identificados durante el año. Esto sugiere que la variabilidad de su demanda no es el único problema: también existe una política de reabastecimiento que no está respondiendo adecuadamente a esa variabilidad.

Los productos **AX** (P003 y P008) son más fáciles de gestionar, ya que combinan un alto valor con una demanda relativamente predecible. Esto permite mantener un control más ajustado del inventario sin necesidad de utilizar grandes niveles de stock de seguridad.

Por otro lado, los productos **CX** (P006 y P007) tienen bajo valor y una demanda estable, por lo que pueden gestionarse mediante reglas de reorden automático sin requerir una atención manual constante.

---

## ANÁLISIS 3 — Forecasting de Demanda

### Contexto

Los productos AY (P001, P002 y P004) presentaron quiebres de stock recurrentes durante 2023.

El siguiente paso fue estimar su demanda para anticipar las necesidades de reabastecimiento y evitar que el problema se repitiera durante el primer trimestre de 2024.

### Metodología

* **Tratamiento de quiebres de stock**: los meses con inventario final igual a 0 (junio, septiembre y diciembre) fueron reconstruidos utilizando el promedio de los meses adyacentes sin quiebre. De esta forma, se buscó representar la demanda que probablemente habría ocurrido en condiciones normales, en lugar de utilizar ventas subregistradas producto del quiebre.

* **Selección de modelo**: el dataset cuenta solamente con 12 meses de información, es decir, un ciclo anual. Por esta razón, no hay suficientes datos para aplicar de manera adecuada modelos como Holt-Winters o SARIMA, que requieren al menos 2-3 ciclos completos. El modelo se seleccionó según el comportamiento observado en cada serie:

  * **P001** → Holt, debido a una tendencia descendente clara.
  * **P002** → SES, al no observarse una tendencia ni estacionalidad clara.
  * **P004** → Holt, debido a una tendencia descendente clara.

* **Validación**: cada modelo se entrenó inicialmente utilizando 9 meses (enero-septiembre) y se evaluó utilizando los 3 meses restantes (octubre-diciembre). Una vez validado el modelo, se volvió a entrenar utilizando los 12 meses completos para generar el forecast final.

### Métricas de validación

| Producto | Modelo | MAPE | MAE  | RMSE | Bias  |
| -------- | ------ | ---- | ---- | ---- | ----- |
| P001     | Holt   | 8.4% | 1.83 | 2.10 | 0.17  |
| P002     | SES    | 5.6% | 1.17 | 1.32 | -0.50 |
| P004     | Holt   | 4.6% | 1.67 | 1.73 | -1.00 |

Los tres modelos obtuvieron un MAPE inferior al 10%, lo que representa un buen nivel de precisión para este análisis de demanda.

El Bias negativo observado en P002 y P004 indica una ligera tendencia de los modelos a subestimar la demanda. Por esta razón, se consideró un margen adicional al momento de definir la recomendación de pedido.

### Forecast enero-marzo 2024

| Período      | P001 | P002 | P004 |
| ------------ | ---- | ---- | ---- |
| Enero 2024   | 22   | 20   | 35   |
| Febrero 2024 | 22   | 20   | 35   |
| Marzo 2024   | 21   | 20   | 34   |

### Recomendación de reabastecimiento

| Producto            | Stock actual (dic 2023) | Demanda pronosticada (Q1 2024) | Unidades a pedir |
| ------------------- | ----------------------- | ------------------------------ | ---------------- |
| P001 Laptop ProBook | 0                       | 65                             | **65**           |
| P002 Monitor 24"    | 0                       | 60                             | **60**           |
| P004 Mouse Optico   | 0                       | 104                            | **104**          |

Los tres productos AY terminaron 2023 con inventario igual a 0, confirmando el problema de abastecimiento identificado en el análisis de working capital.

Por lo tanto, si no se realiza un pedido inicial, el problema de quiebres de stock continuaría durante el primer trimestre de 2024.

---

## ANÁLISIS 4 — Política de Inventario (Q,r) con Simulación Monte Carlo

### Contexto

A partir del forecast obtenido en el análisis anterior y de su error de pronóstico (RMSE), se buscó definir una política de inventario para los productos AY.

La idea fue determinar tanto la **cantidad de pedido (Q)** como el **punto de reorden (r)**, incorporando además el costo asociado a los quiebres de stock, algo que las fórmulas clásicas de EOQ/ROP no consideran directamente.

### Metodología

* **Granularidad mensual**: como el forecast y el RMSE del análisis anterior están expresados en base mensual, la simulación también se realiza mes a mes y no día a día. Esto permite mantener consistencia entre las unidades utilizadas en el forecast y las utilizadas en la simulación.

* **Distribución de demanda**: la demanda se modeló mediante una distribución Normal, utilizando el forecast como media y el RMSE como desviación estándar. Este supuesto es razonable considerando que el Bias de los tres modelos está relativamente cerca de cero. Sin embargo, con solo 12 observaciones históricas no fue posible validarlo estadísticamente.

* **Búsqueda grueso → fino**: los rangos de Q y r se obtuvieron a partir de las fórmulas clásicas de EOQ y ROP, en lugar de definirlos arbitrariamente. Estas fórmulas se utilizaron como punto de partida para construir un rango amplio y posteriormente concentrar la búsqueda alrededor de las mejores soluciones encontradas.

* **Common Random Numbers (CRN)**: todas las combinaciones de (Q,r) fueron evaluadas utilizando la misma secuencia de demanda simulada. De esta manera, las políticas se pueden comparar bajo exactamente las mismas condiciones aleatorias.

* **N óptimo de réplicas**: el número de réplicas se calibró mediante una corrida piloto de 200 réplicas y la fórmula N = (Z·σ/E)², utilizando un margen de error del 1% sobre el costo esperado. De esta forma, el número de réplicas no se eligió arbitrariamente y se pudo controlar el costo computacional sin perder precisión.

### Supuestos operativos

| Parámetro                           | Valor                  | Fuente               |
| ----------------------------------- | ---------------------- | -------------------- |
| Lead time                           | 15 días (0.5 meses)    | Supuesto documentado |
| Tasa de costo de capital/almacenaje | 20% anual              | Supuesto documentado |
| Costo de ordenar                    | $50.000 CLP            | Supuesto documentado |
| Costo de quiebre                    | 30% del costo unitario | Supuesto documentado |

### Resultados — Política óptima por producto

| Producto            | N réplicas | Q óptimo | r óptimo | Costo esperado anual | Nivel de servicio |
| ------------------- | ---------- | -------- | -------- | -------------------- | ----------------- |
| P001 Laptop ProBook | 59         | 47       | 42       | $7.774.628           | 83.2%             |
| P002 Monitor 24"    | 54         | 44       | 37       | $3.085.243           | 83.2%             |
| P004 Mouse Optico   | 30         | 106      | 48       | $707.445             | 83.1%             |

### Hallazgo principal

Ninguna de las combinaciones de (Q,r) evaluadas logró alcanzar el objetivo de 95% de nivel de servicio cuando la simulación parte del inventario real de 0 unidades al cierre de 2023.

Esto permite identificar algo importante: **la política de inventario por sí sola no puede solucionar un déficit de stock que ya existe**.

Primero es necesario realizar el pedido inicial de reabastecimiento calculado en el Análisis 3:

* P001 → 65 unidades.
* P002 → 60 unidades.
* P004 → 104 unidades.

Una vez cubierto ese déficit inicial, la política (Q,r) permite mantener el inventario de forma más estable, alcanzando aproximadamente un 83% de nivel de servicio al menor costo esperado dentro de las combinaciones evaluadas.

### Trabajo futuro

Con una mayor cantidad de datos históricos sería posible profundizar el análisis en varias direcciones:

1. Validar estadísticamente la distribución de demanda mediante un test de bondad de ajuste.
2. Comparar el supuesto de distribución Normal con alternativas como bootstrap de residuos históricos.
3. Reemplazar los lead times y costos asumidos por datos reales de la empresa.
4. Extender la política de inventario al resto del portafolio: ROP clásico (revisión continua) para productos AX y una política de revisión periódica (R,S) para productos BX, BY y CX.

---

## Conclusión

El análisis muestra que el DIO de 6 días no representa necesariamente una operación eficiente. En este caso, el indicador está siendo afectado por los quiebres de stock recurrentes de los productos de mayor valor dentro de la categoría Electrónica.

La segmentación ABC-XYZ permite identificar dónde está concentrado el problema: los productos P001, P002 y P004 fueron clasificados como **AY**, combinando alto valor económico con una demanda variable.

El forecasting permite cuantificar el impacto. Con el inventario en 0 al cierre de 2023, se necesitarían 65 unidades de P001, 60 de P002 y 104 de P004 para cubrir la demanda pronosticada del primer trimestre de 2024.

Finalmente, la simulación Monte Carlo permite ir un paso más allá. Los resultados muestran que una política (Q,r) bien calibrada no puede solucionar por sí sola el déficit inicial de inventario. Primero es necesario cubrir ese déficit y, posteriormente, utilizar una política de reorden que permita controlar los costos y mantener el inventario en niveles adecuados.

En conjunto, los cuatro análisis permiten pasar de identificar el problema a proponer una acción concreta: **cuánto pedir inicialmente y cómo gestionar posteriormente el inventario de los productos más críticos**.

---

## Recomendaciones

1. **Reducir DSO** — implementar descuentos por pronto pago para clientes de la zona Sur y utilizar alertas automáticas para realizar seguimiento de las facturas vencidas.

2. **Ejecutar el pedido de reabastecimiento inicial** — solicitar 65 unidades de P001, 60 de P002 y 104 de P004 antes de enero de 2024 para cubrir la demanda proyectada del Q1 y eliminar el déficit inicial de stock.

3. **Adoptar la política (Q,r) calibrada** — una vez cubierto el déficit inicial, realizar pedidos de 47 unidades para P001, 44 para P002 y 106 para P004 cada vez que el inventario alcance los puntos de reorden de 42, 37 y 48 unidades respectivamente.

4. **Aumentar DPO** — negociar plazos de pago más largos con proveedores clave, especialmente con aquellos que actualmente se pagan en menos de 30 días, con el objetivo de liberar capital de trabajo.

5. **Automatizar la gestión de productos CX** — P006 y P007 pueden gestionarse mediante reorden automático o revisión periódica, permitiendo que el equipo concentre su atención en los productos AY, donde el impacto económico y operativo es mayor.

6. **Ampliar el horizonte de datos** — contar con más de 12 meses de historia permitiría aplicar modelos como Holt-Winters o SARIMA para capturar posibles patrones estacionales, validar estadísticamente la distribución de demanda y extender la política de inventario al resto del portafolio.

---

## Herramientas utilizadas

* **Python** — pandas, numpy, matplotlib, statsmodels, scipy (limpieza de datos, cálculo de KPIs, forecasting y simulación Monte Carlo)
* **Google Colab** — entorno de desarrollo
* **Power BI** — dashboard ejecutivo de 6 páginas

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
<img width="1387" height="784" alt="Politica de inventario Q,r Monte Carlo" src="https://github.com/user-attachments/assets/7b0dfed4-c6f1-4586-8b4e-d73d81cfe516" />

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
│   └── 02_Politica_(Q,r)_para_productos_prioritarios_con_montecarlo.ipynb
├── dashboard/
│   └── Proyecto_kpis.pbix
└── README.md
```
