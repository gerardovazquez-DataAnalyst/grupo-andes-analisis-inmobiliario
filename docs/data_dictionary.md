# Diccionario de datos — Grupo Andes

## hecho_ventas_propiedades (fct_ventas)

Tabla de hechos. Cada fila representa una transacción de venta individual.

| Columna | Tipo | Ejemplo | Descripción |
|---------|------|---------|-------------|
| id_venta | Texto | VTA00001 | Identificador único de la transacción (PK) |
| fecha_venta | Fecha | 2024-03-15 | Fecha en que se cerró la venta |
| id_cliente | Texto | CUST00001 | Clave foránea hacia dim_clientes |
| id_propiedad | Texto | PROP00001 | Clave foránea hacia dim_propiedades |
| ciudad | Texto | Bogotá | Ciudad de la transacción (oculta en el modelo — usar dim_propiedades) |
| precio_venta | Entero | 850000 | Precio final de cierre en moneda local |
| tipo_propiedad | Texto | Casa | Tipo de propiedad vendida (oculta en el modelo — usar dim_propiedades) |
| canal_venta | Texto | Corredor | Canal por el que se concretó la venta (Corredor / Directo) |
| porcentaje_comision | Decimal | 0.035 | Tasa de comisión aplicada a esta transacción (varía entre 1% y 5%) |
| monto_comision | Entero | 29750 | Comisión generada = precio_venta × porcentaje_comision |

**Columnas calculadas añadidas en Power BI:**

| Columna | Tipo | Fórmula DAX | Descripción |
|---------|------|-------------|-------------|
| Primera Compra | Fecha | `CALCULATE(MIN(...), ALLEXCEPT(..., id_cliente))` | Fecha de la primera compra del cliente |
| Mes Cohorte | Texto | `FORMAT(Primera Compra, "YYYY-MM")` | Mes de adquisición del cliente |
| Mes Venta | Texto | `FORMAT(fecha_venta, "YYYY-MM")` | Mes de la transacción actual |
| Periodos Desde Primera Compra | Entero | `DATEDIFF(Primera Compra, fecha_venta, MONTH)` | Meses transcurridos desde la adquisición del cliente (0 = mes de primera compra) |

**Notas:**
- Una misma propiedad puede aparecer en múltiples ventas (reventa). Máximo observado: 7 ventas.
- La granularidad es transacción, no propiedad.
- `fecha_venta` se oculta en el modelo; toda filtración temporal pasa por dim_fecha.

---

## dim_clientes

Tabla de dimensión. Cada fila representa un comprador único.

| Columna | Tipo | Valores únicos | Descripción |
|---------|------|---------------:|-------------|
| id_cliente | Texto | 3,500 | Identificador único del cliente (PK) |
| segmento_comprador | Texto | 3 | Clasificación del comprador |
| pais | Texto | 2 | País del cliente |
| ciudad | Texto | 2 | Ciudad del cliente (distinta de la ciudad de la propiedad en algunos casos) |

**Valores categóricos:**

| Campo | Valores |
|-------|---------|
| segmento_comprador | Primera vez, Alto patrimonio, Inversionista |
| pais | Colombia, Mexico |
| ciudad | Bogotá, Ciudad de México |

**Notas:**
- 3,139 clientes tienen al menos una compra; 361 están registrados pero nunca compraron.
- 2,421 clientes (77.1%) son recurrentes (más de una compra).

---

## dim_propiedades

Tabla de dimensión. Cada fila representa un activo inmobiliario único.

| Columna | Tipo | Valores únicos | Descripción |
|---------|------|---------------:|-------------|
| id_propiedad | Texto | 8,000 | Identificador único de la propiedad (PK) |
| tipo_propiedad | Texto | 3 | Clasificación del inmueble |
| ciudad | Texto | 2 | Ciudad donde se ubica la propiedad |
| barrio | Texto | 20 | Barrio o colonia (10 por ciudad) |
| habitaciones | Entero | — | Número de habitaciones |
| tamano_m2 | Entero | — | Superficie en metros cuadrados |
| precio_publicado | Entero | — | Precio de lista original |
| categoria_propiedad | Texto | — | Categoría adicional de clasificación |

**Valores categóricos:**

| Campo | Valores |
|-------|---------|
| tipo_propiedad | Casa, Departamento, Comercial |
| ciudad | Bogotá, Ciudad de México |

**Notas:**
- 5,223 propiedades han sido vendidas al menos una vez; 2,777 nunca se vendieron.
- 2,310 propiedades se vendieron más de una vez.
- El descuento promedio entre precio_publicado y precio_venta es 7.1% (rango: 2%–12%).

---

## dim_fecha

Tabla de dimensión generada en DAX. Abarca del 2023-01-01 al 2024-12-31.

| Columna | Tipo | Ejemplo | Descripción |
|---------|------|---------|-------------|
| Date | Fecha | 2024-03-15 | Fecha individual (PK, marcada como columna de fecha) |
| Año | Entero | 2024 | Año calendario |
| NumMes | Entero | 3 | Número de mes (1–12), usado para ordenar Mes |
| Mes | Texto | marzo | Nombre del mes en español |
| AñoMes | Texto | 2024-03 | Concatenación año-mes para ejes temporales |
| NumTrim | Entero | 1 | Número de trimestre (1–4) |
| Trimestre | Texto | Q1 | Etiqueta del trimestre |
| AñoTrim | Texto | 2024-Q1 | Concatenación año-trimestre |
| NumDiaSem | Entero | 5 | Número de día de la semana (lunes=1) |
| DiaSemana | Texto | viernes | Nombre del día |

**Configuración obligatoria:**
- Tabla marcada como tabla de fechas con la columna `Date`.
- `Mes` ordenado por `NumMes`.
- `Trimestre` ordenado por `NumTrim`.
- `DiaSemana` ordenado por `NumDiaSem`.

---

## Relaciones del modelo

| Origen (1) | Columna | Destino (*) | Columna | Activa | Dirección |
|------------|---------|-------------|---------|--------|-----------|
| dim_clientes | id_cliente | fct_ventas | id_cliente | Sí | Simple |
| dim_propiedades | id_propiedad | fct_ventas | id_propiedad | Sí | Simple |
| dim_fecha | Date | fct_ventas | fecha_venta | Sí | Simple |
