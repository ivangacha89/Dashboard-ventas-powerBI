# 📏 Documentación de Medidas DAX

---

## 🗂️ Índice

1. [Medidas Base](#1-medidas-base)
2. [Metas y Cumplimiento](#2-metas-y-cumplimiento)
3. [Participación](#3-participación)
4. [Inteligencia de Tiempo](#4-inteligencia-de-tiempo)
5. [Visualización y Formato](#5-visualización-y-formato)
6. [Otras Medidas](#6-otras-medidas)

---

## 1. Medidas Base

Estas medidas son la base del modelo. Todas las demás medidas de ventas se construyen sobre ellas, por lo que cualquier ajuste en este nivel impacta el resto del reporte.

---

### `Ventas Totales`

```dax
Ventas Totales = 
SUMX(
    FactVentas,
    FactVentas[Cantidad] * FactVentas[PrecioUnitario] * (1 - FactVentas[Descuento])
)
```

**¿Para qué se usa?**  
Calcula el monto total de ventas aplicando el descuento a nivel de fila usando `SUMX()`. A diferencia de un `SUM()` simple, esta medida itera sobre cada registro de `FactVentas` y multiplica Cantidad × PrecioUnitario × (1 - Descuento), garantizando que el descuento se aplique correctamente antes de agregar. Es la medida raíz del modelo: alimenta las tarjetas KPI, los gráficos de evolución, el ranking y todas las medidas de cumplimiento y participación.

---

### `Cantidad Vendida`

```dax
Cantidad Vendida = SUM(FactVentas[Cantidad])
```

**¿Para qué se usa?**  
Suma el total de unidades vendidas en el período seleccionado. Se usa en análisis de volumen de ventas y en visuals donde interesa distinguir entre el monto monetario y la cantidad de unidades, por ejemplo al comparar el desempeño de productos por volumen vs por valor.

---

### `Total Transacciones`

```dax
Total Transacciones = COUNTROWS(FactVentas)
```

**¿Para qué se usa?**  
Cuenta el número de filas de la tabla de hechos, es decir, el total de transacciones registradas en el período. Se usa como denominador en el cálculo de `Ticket Promedio` y como indicador de actividad comercial en tarjetas KPI.

---

### `Ticket Promedio`

```dax
Ticket Promedio = 
DIVIDE(
    [Ventas Totales],
    [Total Transacciones],
    0
)
```

**¿Para qué se usa?**  
Calcula el valor promedio por transacción dividiendo las ventas totales entre el número de transacciones. El tercer argumento de `DIVIDE()` retorna `0` si no hay transacciones, evitando errores de división. Es un indicador útil para detectar cambios en el comportamiento de compra: un ticket promedio en caída con volumen estable puede señalar mayor frecuencia de compras de menor valor.

---

## 2. Metas y Cumplimiento

Estas medidas permiten comparar las ventas reales frente a las metas presupuestadas, identificando brechas y el estado de cumplimiento por período y canal.

---

### `Meta Ventas`

```dax
Meta Ventas = 
CALCULATE(
    SUM(MetasVentas[MetaVentas]),
    TREATAS(VALUES(Calendario[Año]), MetasVentas[Año]),
    TREATAS(VALUES(Calendario[Mes_Num]), MetasVentas[Mes_Num]),
    TREATAS(VALUES(Canal[CanalID]), MetasVentas[CanalID])
)
```

**¿Para qué se usa?**  
Recupera la meta de ventas correspondiente al contexto de filtro activo (año, mes y canal), usando `TREATAS()` para crear relaciones virtuales entre la tabla de calendario, la dimensión de canal y la tabla de metas. Esto es necesario porque `MetasVentas` no tiene relaciones directas con `Calendario` ni con `Canal` en el modelo. Es la medida base para todos los cálculos de cumplimiento.

> ⚠️ **Nota:** `TREATAS()` crea una relación virtual en tiempo de ejecución. Si los valores de `Año`, `Mes_Num` o `CanalID` no coinciden exactamente entre tablas, la medida puede retornar resultados inesperados o `BLANK()`.

---

### `Diferencia contra Meta`

```dax
Diferencia contra Meta = [Ventas Totales] - [Meta Ventas]
```

**¿Para qué se usa?**  
Calcula la brecha absoluta entre las ventas reales y la meta del período. Un valor positivo indica que se superó la meta; un valor negativo indica subejecución. Se usa en tarjetas KPI y como insumo para el texto del `Resumen Ejecutivo`.

---

### `Cumplimiento Meta`

```dax
Cumplimiento Meta = 
DIVIDE(
    [Ventas Totales],
    [Meta Ventas],
    0
)
```

**¿Para qué se usa?**  
Expresa qué proporción de la meta fue alcanzada. Un valor de `1.0` (100%) significa cumplimiento exacto; valores superiores a `1.0` indican superación de la meta. Se usa en tarjetas KPI, en el `Resumen Ejecutivo` y como base para la medida `Estado KPI`.

---

### `Cumplimiento por Canal`

```dax
Cumplimiento por Canal = 
DIVIDE(
    [Ventas Totales],
    CALCULATE(
        SUMX(MetasVentas, MetasVentas[MetaVentas]),
        ALLEXCEPT(MetasVentas, MetasVentas[CanalID])
    ),
    0
)
```

**¿Para qué se usa?**  
Calcula el cumplimiento de meta específicamente a nivel de canal, manteniendo el filtro por `CanalID` pero eliminando cualquier otro filtro activo sobre `MetasVentas` mediante `ALLEXCEPT()`. Permite comparar el desempeño de cada canal de venta contra su propia meta, independientemente de los filtros de período aplicados en el reporte.

---

### `Estado KPI`

```dax
Estado KPI = 
SWITCH(
    TRUE(),
    [Cumplimiento Meta] >= 1, "Cumple",
    [Cumplimiento Meta] >= 0.9, "En riesgo",
    "No cumple"
)
```

**¿Para qué se usa?**  
Convierte el valor numérico de `Cumplimiento Meta` en una etiqueta de estado para facilitar la lectura ejecutiva. La lógica de clasificación es:

| Rango | Estado | Significado |
|---|---|---|
| ≥ 100% | `Cumple` | Meta alcanzada o superada |
| ≥ 90% y < 100% | `En riesgo` | Cerca de la meta pero sin alcanzarla |
| < 90% | `No cumple` | Brecha significativa respecto a la meta |

Se usa en formato condicional, tarjetas de estado y visuals de semáforo en el dashboard.

---

## 3. Participación

Estas medidas calculan el peso relativo de cada segmento (canal, categoría, cliente, producto) sobre el total de ventas, permitiendo identificar dónde se concentra el volumen de negocio.

---

### `Participacion Canal`

```dax
Participacion Canal = 
DIVIDE(
    [Ventas Totales],
    CALCULATE([Ventas Totales], ALL(Canal)),
    0
)
```

**¿Para qué se usa?**  
Calcula qué porcentaje del total de ventas corresponde a cada canal. `ALL(Canal)` elimina los filtros sobre la dimensión de canal para obtener el denominador global. Se usa en gráficos de torta o barras apiladas para mostrar la distribución de ventas por canal.

---

### `Participacion Categoria`

```dax
Participacion Categoria = 
DIVIDE(
    [Ventas Totales],
    CALCULATE([Ventas Totales], ALL(Producto[Categoria])),
    0
)
```

**¿Para qué se usa?**  
Calcula el peso de cada categoría de producto sobre el total de ventas. A diferencia de `ALL(Producto)`, aquí se elimina el filtro únicamente sobre la columna `Categoria`, manteniendo otros filtros activos sobre la dimensión de producto. Se usa en análisis de mix de producto por categoría.

---

### `Participacion Cliente`

```dax
Participacion Cliente = 
DIVIDE(
    [Ventas Totales],
    CALCULATE([Ventas Totales], ALL(Cliente)),
    0
)
```

**¿Para qué se usa?**  
Calcula la participación de cada cliente sobre el total de ventas. `ALL(Cliente)` elimina todos los filtros sobre la dimensión de clientes. Se usa en el ranking de clientes para mostrar, además del monto, qué porcentaje del negocio representa cada uno.

---

### `Participacion Ventas`

```dax
Participacion Ventas = 
DIVIDE(
    [Ventas Totales],
    CALCULATE([Ventas Totales], ALL(Producto)),
    0
)
```

**¿Para qué se usa?**  
Calcula la participación de cada producto sobre el total de ventas, eliminando todos los filtros sobre la dimensión de producto. A diferencia de `Participacion Categoria`, esta medida opera a nivel de producto individual. Se usa en el ranking de productos para contextualizar el peso de cada SKU dentro del portafolio.

---

## 4. Inteligencia de Tiempo

Medidas que usan funciones de inteligencia de tiempo de DAX para análisis de tendencias, comparaciones contra períodos anteriores y acumulados.

---

### `Ventas Periodo Anterior`

```dax
Ventas Periodo Anterior = 
CALCULATE(
    [Ventas Totales],
    PREVIOUSMONTH(Calendario[Date])
)
```

**¿Para qué se usa?**  
Calcula las ventas totales del mes inmediatamente anterior al período seleccionado, usando la tabla `Calendario` como referencia. Requiere que el modelo tenga una tabla de calendario con relación activa a la tabla de hechos. Es la base para calcular `Variacion Ventas` y `Variacion %`.

---

### `Variacion Ventas`

```dax
Variacion Ventas = [Ventas Totales] - [Ventas Periodo Anterior]
```

**¿Para qué se usa?**  
Calcula la diferencia absoluta entre las ventas del período actual y las del mes anterior. Un valor positivo indica crecimiento; un valor negativo indica caída. Es la base para `Variacion %` y se referencia en el `Resumen Ejecutivo`.

---

### `Variacion %`

```dax
Variacion % = 
DIVIDE(
    [Variacion Ventas],
    [Ventas Periodo Anterior],
    0
)
```

**¿Para qué se usa?**  
Expresa en porcentaje el cambio de ventas respecto al mes anterior. Se usa en tarjetas KPI para mostrar la tendencia de crecimiento o decrecimiento mensual, y se incorpora al texto del `Resumen Ejecutivo` para comunicar la evolución de manera narrativa.

---

### `Ventas Acumuladas`

```dax
Ventas Acumuladas = TOTALYTD([Ventas Totales], Calendario[Date])
```

**¿Para qué se usa?**  
Acumula las ventas totales desde el inicio del año hasta la fecha seleccionada (*Year-To-Date*). Permite ver el avance acumulado en lo que va del año, útil para gráficos de evolución mensual donde se quiere mostrar el progreso acumulado en lugar del valor puntual del mes. Facilita la comparación del YTD actual contra la meta anual.

---

## 5. Visualización y Formato

Medidas orientadas a controlar cómo se presenta la información en las visualizaciones del dashboard.

---

### `Resumen Ejecutivo`

```dax
Resumen Ejecutivo = 
"Las ventas fueron " & FORMAT([Ventas Totales], "$ #,##0") &
", con una meta de " & FORMAT([Meta Ventas], "$ #,##0") &
". El cumplimiento fue de " & FORMAT([Cumplimiento Meta], "0.0%") &
" y la variación frente al periodo anterior fue de " & FORMAT([Variacion %], "0.0%") & "."
```

**¿Para qué se usa?**  
Genera automáticamente un párrafo narrativo que resume los indicadores clave del período seleccionado: ventas reales, meta, cumplimiento y variación mensual. Se actualiza dinámicamente con cada interacción de filtro (año, mes, canal). Permite que cualquier usuario entienda el estado del negocio sin necesidad de interpretar los gráficos.

**Ejemplo de salida:**  
> *"Las ventas fueron $1,450,000, con una meta de $1,200,000. El cumplimiento fue de 120.8% y la variación frente al periodo anterior fue de 8.3%."*

---

## 6. Otras Medidas

---

### `Ranking Clientes`

```dax
Ranking Clientes = 
IF(
    ISINSCOPE(Cliente[Cliente]),
    RANKX(
        ALL(Cliente[Cliente]),
        [Ventas Totales],
        ,
        DESC,
        Dense
    ),
    BLANK()
)
```

**¿Para qué se usa?**  
Asigna un ranking numérico a cada cliente según sus ventas totales de mayor a menor. El uso de `ISINSCOPE()` es clave: hace que el ranking solo aparezca cuando la visual está desglosada por cliente, devolviendo `BLANK()` en totales o subtotales para evitar números sin sentido. Se usa en la visual de ranking de clientes para identificar los de mayor contribución al negocio.

---

### `Ranking Productos`

```dax
Ranking Productos = 
IF(
    ISINSCOPE(Producto[Producto]),
    RANKX(
        ALL(Producto[Producto]),
        [Ventas Totales],
        ,
        DESC,
        Dense
    ),
    BLANK()
)
```

**¿Para qué se usa?**  
Asigna un ranking numérico a cada producto según sus ventas totales de mayor a menor, aplicando la misma lógica de `ISINSCOPE()` que `Ranking Clientes`. Se usa en la visual de detalle y ranking de productos para identificar los SKUs más relevantes del portafolio.

---

## 📌 Dependencias del Modelo

Para que estas medidas funcionen correctamente, el modelo debe tener las siguientes tablas y relaciones configuradas:

| Tabla | Tipo | Descripción |
|---|---|---|
| `FactVentas` | Fact | Tabla de hechos con las transacciones de ventas (Cantidad, PrecioUnitario, Descuento) |
| `Cliente` | Dimensión | Información y segmentación de clientes |
| `Producto` | Dimensión | Catálogo de productos con categorías estandarizadas |
| `Canal` | Dimensión | Canales de venta con `CanalID` para cruce con metas |
| `MetasVentas` | Metas | Metas de ventas por año, mes y canal (relacionada vía `TREATAS`) |
| `Calendario` | Dimensión de tiempo | Tabla de fechas con columnas `Date`, `Año`, `Mes_Num` |

---

*Documentación generada para el repositorio del proyecto — Power BI Dashboard de Ventas*
