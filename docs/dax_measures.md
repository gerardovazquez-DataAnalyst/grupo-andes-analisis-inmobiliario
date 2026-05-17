# Medidas DAX — Grupo Andes

Todas las medidas viven en la tabla `_Medidas` (tabla vacía creada como contenedor organizacional).
Las columnas calculadas viven en `fct_ventas`.

## Organización por carpetas de visualización

Las medidas están agrupadas en 8 carpetas usando la propiedad **Display Folder** de Power BI:

| Carpeta | Contenido |
|---------|-----------|
| `01_Base` | Medidas fundamentales: Ingreso Total, Cantidad de Ventas, Ticket Promedio, Comisión Total, % Comisión Efectiva |
| `02_Deltas` | Comparativos dinámicos año vs año: Delta % Ingreso/Ventas/Ticket/Comisión |
| `03_Etiquetas` | Medidas de texto con flecha y signo para Reference Labels |
| `04_Participación` | % Part. Tipo Propiedad, Canal Venta, Segmento |
| `05_Volumen` | % Volumen Tipo Propiedad, Canal, Segmento |
| `06_Tiempo` | Inteligencia temporal: Ventas YTD, Ventas MTD |
| `07_Comparativos` | Medidas para visuals condicionales: Ingreso 2023/2024 Solo Ambos, Titulo Inferior |
| `08_Cohortes` | Clientes en Cohorte, Tasa de Retención |

El prefijo numérico fuerza el orden en el panel de campos. Sin él, Power BI las ordena alfabéticamente.

---

## 4.1 Medidas base

```dax
Ingreso Total = 
SUM ( fct_ventas[precio_venta] )
```
Formato: Moneda, 0 decimales.

```dax
Cantidad de Ventas = 
COUNTROWS ( fct_ventas )
```
Formato: Número entero.

```dax
Ticket Promedio = 
DIVIDE ( [Ingreso Total], [Cantidad de Ventas], 0 )
```
Formato: Moneda, 0 decimales.

```dax
Comisión Total = 
SUM ( fct_ventas[monto_comision] )
```
Formato: Moneda, 0 decimales.

```dax
% Comisión Efectiva = 
DIVIDE ( [Comisión Total], [Ingreso Total], 0 )
```
Formato: Porcentaje, 2 decimales.

---

## 4.1b Deltas dinámicos para tarjetas KPI

Comparan el año seleccionado contra el otro año del dataset. Si ambos años están seleccionados, devuelven BLANK (el delta desaparece). Usan `SELECTEDVALUE` para detectar si hay exactamente un año en el filtro y `REMOVEFILTERS` para calcular el denominador sin el filtro del slicer.

```dax
Delta % Ingreso = 
VAR _añoSel = SELECTEDVALUE ( dim_fecha[Año] )
VAR _ingresoActual = [Ingreso Total]
VAR _añoComparar = 
    IF ( _añoSel = 2024, 2023, IF ( _añoSel = 2023, 2024 ) )
VAR _ingresoComparar = 
    CALCULATE (
        [Ingreso Total],
        REMOVEFILTERS ( dim_fecha[Año] ),
        dim_fecha[Año] = _añoComparar
    )
RETURN
    IF (
        NOT ISBLANK ( _añoSel ),
        DIVIDE ( _ingresoActual - _ingresoComparar, _ingresoComparar, 0 ),
        BLANK ()
    )
```

```dax
Delta % Ventas = 
VAR _añoSel = SELECTEDVALUE ( dim_fecha[Año] )
VAR _actual = [Cantidad de Ventas]
VAR _añoComp = IF ( _añoSel = 2024, 2023, IF ( _añoSel = 2023, 2024 ) )
VAR _comparar = 
    CALCULATE ( [Cantidad de Ventas], REMOVEFILTERS ( dim_fecha[Año] ), dim_fecha[Año] = _añoComp )
RETURN
    IF ( NOT ISBLANK ( _añoSel ), DIVIDE ( _actual - _comparar, _comparar, 0 ), BLANK () )
```

```dax
Delta % Ticket = 
VAR _añoSel = SELECTEDVALUE ( dim_fecha[Año] )
VAR _actual = [Ticket Promedio]
VAR _añoComp = IF ( _añoSel = 2024, 2023, IF ( _añoSel = 2023, 2024 ) )
VAR _comparar = 
    CALCULATE ( [Ticket Promedio], REMOVEFILTERS ( dim_fecha[Año] ), dim_fecha[Año] = _añoComp )
RETURN
    IF ( NOT ISBLANK ( _añoSel ), DIVIDE ( _actual - _comparar, _comparar, 0 ), BLANK () )
```

```dax
Delta % Comisión = 
VAR _añoSel = SELECTEDVALUE ( dim_fecha[Año] )
VAR _actual = [Comisión Total]
VAR _añoComp = IF ( _añoSel = 2024, 2023, IF ( _añoSel = 2023, 2024 ) )
VAR _comparar = 
    CALCULATE ( [Comisión Total], REMOVEFILTERS ( dim_fecha[Año] ), dim_fecha[Año] = _añoComp )
RETURN
    IF ( NOT ISBLANK ( _añoSel ), DIVIDE ( _actual - _comparar, _comparar, 0 ), BLANK () )
```
Formato: Porcentaje, 1 decimal. Formato condicional de color: verde `#1D9E75` si ≥ 0, coral `#D85A30` si < 0.

**Nota:** en DAX, las variables (`VAR`) no aceptan la ñ. Usar `_anioSel` en lugar de `_añoSel`.

### Etiquetas de texto para tarjetas KPI

Devuelven texto con flecha + porcentaje formateado (▲ +11.1% o ▼ -10.0%). Se usan como Reference Label en Card (new).

```dax
Etiqueta Delta Ingreso = 
VAR _delta = [Delta % Ingreso]
RETURN
    IF ( ISBLANK(_delta), "",
        IF ( _delta >= 0, "▲ +" & FORMAT(_delta, "0.0%"),
            "▼ " & FORMAT(_delta, "0.0%") ) )
```

Mismo patrón para `Etiqueta Delta Ventas`, `Etiqueta Delta Ticket`, `Etiqueta Delta Comision`.

### Título dinámico y visuals condicionales

```dax
Titulo Inferior = 
IF (
    ISBLANK ( SELECTEDVALUE ( dim_fecha[Año] ) ),
    "Ingreso por tipo de propiedad · 2023 vs 2024",
    "Crecimiento YoY por tipo de propiedad · " & SELECTEDVALUE ( dim_fecha[Año] )
)
```

```dax
Ingreso 2023 Solo Ambos = 
IF (
    ISBLANK ( SELECTEDVALUE ( dim_fecha[Año] ) ),
    CALCULATE ( [Ingreso Total], dim_fecha[Año] = 2023 ),
    BLANK ()
)
```

```dax
Ingreso 2024 Solo Ambos = 
IF (
    ISBLANK ( SELECTEDVALUE ( dim_fecha[Año] ) ),
    CALCULATE ( [Ingreso Total], dim_fecha[Año] = 2024 ),
    BLANK ()
)
```

Técnica de visuals apilados: Visual A (barras YoY) y Visual B (barras agrupadas 2023 vs 2024) comparten la misma posición. Cuando uno tiene datos, el otro devuelve BLANK y se vuelve transparente.

---

## 4.2 Medidas de participación

Usan `CALCULATE` + `ALL` para remover el filtro de una dimensión en el denominador, manteniendo el filtro en el numerador.

```dax
% Part. Tipo Propiedad = 
DIVIDE (
    [Ingreso Total],
    CALCULATE ( [Ingreso Total], ALL ( dim_propiedades[tipo_propiedad] ) ),
    0
)
```

```dax
% Part. Canal Venta = 
DIVIDE (
    [Ingreso Total],
    CALCULATE ( [Ingreso Total], ALL ( fct_ventas[canal_venta] ) ),
    0
)
```

```dax
% Part. Segmento = 
DIVIDE (
    [Ingreso Total],
    CALCULATE ( [Ingreso Total], ALL ( dim_clientes[segmento_comprador] ) ),
    0
)
```
Formato: Porcentaje, 1 decimal.

---

## 4.3 Inteligencia de tiempo

Requieren que `dim_fecha` esté marcada como tabla de fechas.

```dax
Ventas YTD = 
TOTALYTD ( [Ingreso Total], dim_fecha[Date] )
```

```dax
Ventas MTD = 
TOTALMTD ( [Ingreso Total], dim_fecha[Date] )
```

```dax
Ingreso Año Anterior = 
CALCULATE (
    [Ingreso Total],
    SAMEPERIODLASTYEAR ( dim_fecha[Date] )
)
```

```dax
Crecimiento YoY = 
VAR _actual = [Ingreso Total]
VAR _anterior = [Ingreso Año Anterior]
RETURN
    DIVIDE ( _actual - _anterior, _anterior, 0 )
```
Formato: YTD y Año Anterior → Moneda. YoY → Porcentaje, 1 decimal.

---

## 4.4 Columnas calculadas (cohortes)

Creadas en la tabla `fct_ventas` como columnas calculadas (no como medidas).

```dax
Primera Compra = 
CALCULATE (
    MIN ( fct_ventas[fecha_venta] ),
    ALLEXCEPT ( fct_ventas, fct_ventas[id_cliente] )
)
```

```dax
Mes Cohorte = 
FORMAT ( fct_ventas[Primera Compra], "YYYY-MM" )
```

```dax
Mes Venta = 
FORMAT ( fct_ventas[fecha_venta], "YYYY-MM" )
```

```dax
Periodos Desde Primera Compra = 
DATEDIFF (
    fct_ventas[Primera Compra],
    fct_ventas[fecha_venta],
    MONTH
)
```
Normaliza el eje temporal: 0 = mes de adquisición, 1 = mes siguiente, etc. Permite comparar cohortes entre sí.

### Medidas de retención (en tabla _Medidas)

```dax
Clientes en Cohorte = 
CALCULATE (
    DISTINCTCOUNT ( fct_ventas[id_cliente] ),
    FILTER (
        ALL ( fct_ventas ),
        fct_ventas[Mes Cohorte] = MAX ( fct_ventas[Mes Cohorte] )
            && fct_ventas[Periodos Desde Primera Compra] = 0
    )
)
```
Total de clientes únicos en el periodo 0 de cada cohorte. Es el denominador fijo para retención.

```dax
Tasa de Retención = 
DIVIDE (
    DISTINCTCOUNT ( fct_ventas[id_cliente] ),
    [Clientes en Cohorte],
    0
)
```
Formato: Porcentaje, 1 decimal. Aplicar formato condicional de escala de color (verde oscuro → blanco) en la matriz.

### Medidas de adquisición e ingreso nuevo vs recurrente

```dax
Clientes Nuevos = 
CALCULATE (
    DISTINCTCOUNT ( fct_ventas[id_cliente] ),
    fct_ventas[Periodos Desde Primera Compra] = 0
)
```
Formato: Número entero. Cuenta clientes únicos cuya transacción es su primera compra.

```dax
Tasa Recurrencia = 
VAR _totalClientes = DISTINCTCOUNT ( fct_ventas[id_cliente] )
VAR _recurrentes = 
    CALCULATE (
        DISTINCTCOUNT ( fct_ventas[id_cliente] ),
        FILTER (
            VALUES ( fct_ventas[id_cliente] ),
            CALCULATE ( COUNTROWS ( fct_ventas ) ) > 1
        )
    )
RETURN
    DIVIDE ( _recurrentes, _totalClientes, 0 )
```
Formato: Porcentaje, 1 decimal.

```dax
Compras por Cliente = 
DIVIDE (
    COUNTROWS ( fct_ventas ),
    DISTINCTCOUNT ( fct_ventas[id_cliente] ),
    0
)
```
Formato: Decimal, 1 decimal.

```dax
Ingreso Primera Compra = 
CALCULATE (
    [Ingreso Total],
    fct_ventas[Periodos Desde Primera Compra] = 0
)
```

```dax
Ingreso Recurrente = 
CALCULATE (
    [Ingreso Total],
    fct_ventas[Periodos Desde Primera Compra] > 0
)
```

```dax
% Ingreso Primera Compra = 
DIVIDE ( [Ingreso Primera Compra], [Ingreso Total], 0 )
```

```dax
% Ingreso Recurrente = 
DIVIDE ( [Ingreso Recurrente], [Ingreso Total], 0 )
```
Formato de ingreso: Moneda. Formato de porcentajes: Porcentaje, 1 decimal.

---

## Verificación rápida

Para validar que las medidas funcionan correctamente:

1. Crear una tabla con `dim_propiedades[tipo_propiedad]` + `[Ingreso Total]` + `[% Part. Tipo Propiedad]` → la columna de participación debe sumar ~100%.
2. Crear una tarjeta con `[Crecimiento YoY]` + slicer de Año = 2024 → debe mostrar ~+11.1%.
3. Filtrar `fct_ventas` por un cliente específico → todas las filas deben tener el mismo `Mes Cohorte`.
