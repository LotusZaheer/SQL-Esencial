# PostgreSQL vs BigQuery - Guía Práctica

## Introducción

PostgreSQL y BigQuery resuelven problemas diferentes pero con SQL fundamentalmente similar. Esta guía te ayudará a entender las diferencias.

---

## 1. Diferencias Fundamentales

### PostgreSQL
```
✓ Base de datos relacional transaccional
✓ Perfecta para aplicaciones OLTP (Online Transaction Processing)
✓ Garantías ACID completas
✓ Ideal para datos que cambian constantemente
✓ Buena para < 10TB en una máquina
✓ Licencia: Open Source (PostgreSQL License)
✓ Local o en servidor propio
```

### BigQuery
```
✓ Data warehouse serverless (análitica masiva)
✓ Perfecta para OLAP (Online Analytical Processing)
✓ Procesamiento de 100s de TB sin problema
✓ Ideal para análisis históricos, no actualización continua
✓ Se paga por datos escaneados
✓ Licencia: Propiedad de Google
✓ Solo en la nube
```

**En resumen:**
- **PostgreSQL**: Aplicaciones, transacciones, datos en tiempo real
- **BigQuery**: Análisis masivos, reportes, preguntas complejas sobre volúmenes enormes

---

## 2. Sintaxis Comparativa

### 2.1 - Funciones de Fecha

#### PostgreSQL
```sql
-- Formato PostgreSQL
SELECT
    DATE_TRUNC('month', fecha) AS mes,
    EXTRACT(YEAR FROM fecha) AS año,
    AGE(CURRENT_DATE, fecha_nacimiento) AS edad_calculada
FROM usuarios;
```

#### BigQuery
```sql
-- Formato BigQuery
SELECT
    DATE_TRUNC(fecha, MONTH) AS mes,
    EXTRACT(YEAR FROM fecha) AS año,
    DATE_DIFF(CURRENT_DATE(), fecha_nacimiento, YEAR) AS edad_calculada
FROM `proyecto.dataset.usuarios`;
```

**Diferencias clave:**
- BigQuery usa `DATE_TRUNC(columna, intervalo)` en lugar de `DATE_TRUNC('intervalo', columna)`
- BigQuery usa `DATE_DIFF` en lugar de `AGE`
- BigQuery entra las tablas entre backticks: `` `proyecto.dataset.tabla` ``

---

### 2.2 - Funciones de Texto

#### PostgreSQL
```sql
SELECT
    UPPER(nombre) AS mayusculas,
    LOWER(nombre) AS minusculas,
    LENGTH(nombre) AS largo,
    SUBSTR(nombre, 1, 5) AS primeros_5,
    SPLIT_PART(email, '@', 2) AS dominio,
    TRIM(nombre) AS sin_espacios_extremos
FROM usuarios;
```

#### BigQuery
```sql
SELECT
    UPPER(nombre) AS mayusculas,
    LOWER(nombre) AS minusculas,
    LENGTH(nombre) AS largo,
    SUBSTR(nombre, 1, 5) AS primeros_5,
    SPLIT_STRING(email, '@')[OFFSET(1)] AS dominio,
    TRIM(nombre) AS sin_espacios_extremos
FROM `proyecto.dataset.usuarios`;
```

**Diferencias clave:**
- BigQuery: `SPLIT_STRING` en lugar de `SPLIT_PART`, con `[OFFSET(n)]` para acceso
- BigQuery: Los índices en arrays empiezan en 0 (como muchos lenguajes)

---

### 2.3 - Funciones Matemáticas

#### PostgreSQL
```sql
SELECT
    ROUND(valor, 2) AS redondeado,
    CEIL(valor) AS arriba,
    FLOOR(valor) AS abajo,
    POWER(valor, 2) AS al_cuadrado,
    SQRT(valor) AS raiz
FROM transacciones;
```

#### BigQuery
```sql
SELECT
    ROUND(valor, 2) AS redondeado,
    CEIL(valor) AS arriba,
    FLOOR(valor) AS abajo,
    POWER(valor, 2) AS al_cuadrado,
    SQRT(valor) AS raiz
FROM `proyecto.dataset.transacciones`;
```

**Aquí son casi idénticas,** la única diferencia es el nombre de tabla en BigQuery.

---

### 2.4 - Expresiones Regulares

#### PostgreSQL
```sql
-- Validar email
SELECT
    email
FROM usuarios
WHERE email ~ '.*@gmail\.com$';  -- ~ es SIMILAR TO (regex)

-- Extraer números
SELECT
    nombre,
    (regexp_matches(nombre, '\d+', 'g'))[1] AS numero
FROM usuarios
WHERE nombre ~ '\d+';
```

#### BigQuery
```sql
-- Validar email
SELECT
    email
FROM `proyecto.dataset.usuarios`
WHERE email REGEXP r'^[a-zA-Z0-9._%+-]+@gmail\.com$';

-- Extraer números
SELECT
    nombre,
    REGEXP_EXTRACT(nombre, r'\d+') AS numero
FROM `proyecto.dataset.usuarios`
WHERE REGEXP_CONTAINS(nombre, r'\d+');
```

**Diferencias clave:**
- PostgreSQL: `~` para matching, `regexp_matches()` para extracción
- BigQuery: `REGEXP_CONTAINS()` para validación, `REGEXP_EXTRACT()` para extracción
- BigQuery usa raw strings `r'...'` para expresiones regulares

---

## 3. Window Functions (Muy similares)

### PostgreSQL
```sql
SELECT
    nombre,
    ingresos,
    ROW_NUMBER() OVER (ORDER BY ingresos DESC) AS ranking,
    SUM(ingresos) OVER (ORDER BY ingresos DESC) AS suma_acumulada
FROM usuarios;
```

### BigQuery
```sql
SELECT
    nombre,
    ingresos,
    ROW_NUMBER() OVER (ORDER BY ingresos DESC) AS ranking,
    SUM(ingresos) OVER (ORDER BY ingresos DESC) AS suma_acumulada
FROM `proyecto.dataset.usuarios`;
```

**La sintaxis es prácticamente idéntica.** BigQuery hace poco tiempo añadió soporte completo para window functions.

---

## 4. CTEs y Subconsultas

### PostgreSQL
```sql
WITH usuarios_activos AS (
    SELECT id, nombre, ingresos
    FROM usuarios
    WHERE estado = 'activo'
)
SELECT * FROM usuarios_activos WHERE ingresos > 50000;
```

### BigQuery
```sql
WITH usuarios_activos AS (
    SELECT id, nombre, ingresos
    FROM `proyecto.dataset.usuarios`
    WHERE estado = 'activo'
)
SELECT * FROM usuarios_activos WHERE ingresos > 50000;
```

**Prácticamente idénticas.** La única diferencia es la notación de tabla en BigQuery.

---

## 5. Tipos de Datos

### PostgreSQL - Tipos comunes
```sql
INTEGER/INT              -- Números enteros
DECIMAL(10,2)           -- Números precisos (para dinero)
VARCHAR(100)            -- Texto variable hasta 100 caracteres
TEXT                    -- Texto sin límite
DATE                    -- Solo fecha
TIMESTAMP               -- Fecha y hora
BOOLEAN                 -- true/false
ARRAY                   -- Arrays: ARRAY[1,2,3]
JSON/JSONB              -- Datos JSON
```

### BigQuery - Tipos comunes
```
INT64                   -- Números enteros (equivalente a INTEGER)
NUMERIC(38,9)           -- Números precisos (para dinero)
STRING                  -- Texto (equivalente a VARCHAR)
DATE                    -- Solo fecha
TIMESTAMP               -- Fecha y hora (UTC)
BOOL                    -- true/false
ARRAY<T>                -- Arrays: ARRAY[1, 2, 3]
JSON                    -- Datos JSON (STRUCT es más usado)
STRUCT                  -- Tipo compuesto
```

---

## 6. Casos de Uso Reales

### Caso 1: Análisis de Series Temporales

#### PostgreSQL
```sql
-- Crecimiento mes a mes
WITH ventas_mensuales AS (
    SELECT
        DATE_TRUNC('month', fecha)::DATE AS mes,
        SUM(monto) AS total
    FROM transacciones
    GROUP BY DATE_TRUNC('month', fecha)
)
SELECT
    mes,
    total,
    LAG(total) OVER (ORDER BY mes) AS mes_anterior,
    ROUND(((total - LAG(total) OVER (ORDER BY mes))
        / LAG(total) OVER (ORDER BY mes) * 100), 2) AS crecimiento_pct
FROM ventas_mensuales
ORDER BY mes DESC;
```

#### BigQuery
```sql
-- Crecimiento mes a mes
WITH ventas_mensuales AS (
    SELECT
        DATE_TRUNC(fecha, MONTH) AS mes,
        SUM(monto) AS total
    FROM `proyecto.dataset.transacciones`
    GROUP BY DATE_TRUNC(fecha, MONTH)
)
SELECT
    mes,
    total,
    LAG(total) OVER (ORDER BY mes) AS mes_anterior,
    ROUND(((total - LAG(total) OVER (ORDER BY mes))
        / LAG(total) OVER (ORDER BY mes) * 100), 2) AS crecimiento_pct
FROM ventas_mensuales
ORDER BY mes DESC;
```

**La lógica es idéntica**, solo cambios en fecha y naming de tabla.

---

### Caso 2: Segmentación de Clientes

#### PostgreSQL
```sql
SELECT
    id,
    nombre,
    SUM(monto) AS total_gasto,
    COUNT(DISTINCT EXTRACT(DATE FROM fecha)) AS dias_compra,
    MAX(fecha) AS ultima_compra,
    NTILE(4) OVER (ORDER BY SUM(monto)) AS cuartil_gasto
FROM transacciones t
JOIN usuarios u ON t.usuario_id = u.id
GROUP BY u.id, u.nombre
HAVING COUNT(*) >= 3
ORDER BY cuartil_gasto DESC;
```

#### BigQuery
```sql
SELECT
    id,
    nombre,
    SUM(monto) AS total_gasto,
    COUNT(DISTINCT CAST(fecha AS DATE)) AS dias_compra,
    MAX(fecha) AS ultima_compra,
    NTILE(4) OVER (ORDER BY SUM(monto)) AS cuartil_gasto
FROM `proyecto.dataset.transacciones` t
JOIN `proyecto.dataset.usuarios` u ON t.usuario_id = u.id
GROUP BY u.id, u.nombre
HAVING COUNT(*) >= 3
ORDER BY cuartil_gasto DESC;
```

---

## 7. Optimización: BigQuery Específico

### Costos en BigQuery

BigQuery te cobra por **datos escaneados**, no por tiempo o CPU.

```sql
-- ❌ MALO: Escanea toda la tabla
SELECT * FROM `proyecto.dataset.usuarios`;

-- ✓ BUENO: Selecciona solo columnas necesarias
SELECT nombre, email, ciudad FROM `proyecto.dataset.usuarios`;

-- ✓ MEJOR: Filtra por partición si existe
SELECT nombre, email FROM `proyecto.dataset.usuarios`
WHERE DATE(fecha_registro) >= '2024-01-01';
```

### Tablas Particionadas

```sql
-- BigQuery: crear tabla particionada por fecha
CREATE OR REPLACE TABLE `proyecto.dataset.transacciones`
PARTITION BY DATE(fecha)
AS
SELECT * FROM `proyecto.dataset.transacciones_temp`;

-- Esto hace que los filtros de fecha sean mucho más baratos
SELECT * FROM `proyecto.dataset.transacciones`
WHERE fecha >= '2024-01-01';  -- Solo escanea enero en adelante
```

### STRUCT y ARRAY (mejor que JSON en BigQuery)

```sql
-- Ejemplo: orden con múltiples items
SELECT
    orden.id,
    orden.cliente,
    item.nombre,
    item.cantidad,
    item.precio
FROM `proyecto.dataset.ordenes`,
UNNEST(ordenes.items) AS item;

-- Construcción de STRUCT
SELECT
    id,
    nombre,
    STRUCT(
        apellido,
        email,
        ciudad
    ) AS contacto
FROM `proyecto.dataset.usuarios`;
```

---

## 8. Migración de PostgreSQL a BigQuery

### Paso 1: Exportar datos de PostgreSQL
```sql
-- En PostgreSQL, exportar a CSV
\COPY (SELECT * FROM usuarios) TO '/tmp/usuarios.csv' WITH (FORMAT CSV, HEADER);
```

### Paso 2: Importar a BigQuery
```sql
-- En BigQuery Cloud Shell o mediante UI
bq load --source_format=CSV \
  proyecto.dataset.usuarios \
  /tmp/usuarios.csv \
  id:INTEGER,nombre:STRING,email:STRING,fecha_registro:TIMESTAMP
```

### Paso 3: Adaptar las queries

**PostgreSQL original:**
```sql
SELECT
    DATE_TRUNC('month', fecha_registro) AS mes,
    UPPER(nombre) AS nombre_upper,
    ROUND(SUM(monto), 2) AS total
FROM usuario_transacciones
GROUP BY DATE_TRUNC('month', fecha_registro), UPPER(nombre);
```

**BigQuery adaptado:**
```sql
SELECT
    DATE_TRUNC(fecha_registro, MONTH) AS mes,
    UPPER(nombre) AS nombre_upper,
    ROUND(SUM(monto), 2) AS total
FROM `proyecto.dataset.usuario_transacciones`
GROUP BY DATE_TRUNC(fecha_registro, MONTH), UPPER(nombre);
```

---

## 9. Cheat Sheet Rápido

| Funcionalidad | PostgreSQL | BigQuery |
|---|---|---|
| Truncar fecha | `DATE_TRUNC('month', col)` | `DATE_TRUNC(col, MONTH)` |
| Extraer parte de fecha | `EXTRACT(YEAR FROM col)` | `EXTRACT(YEAR FROM col)` |
| Edad | `AGE(fecha1, fecha2)` | `DATE_DIFF(fecha1, fecha2, YEAR)` |
| Texto mayúscula | `UPPER(col)` | `UPPER(col)` |
| Largo de texto | `LENGTH(col)` | `LENGTH(col)` |
| Substring | `SUBSTR(col, 1, 5)` | `SUBSTR(col, 1, 5)` |
| Dividir string | `SPLIT_PART(col, ',', 1)` | `SPLIT_STRING(col, ',')[OFFSET(0)]` |
| Regex match | `col ~ 'pattern'` | `REGEXP_CONTAINS(col, r'pattern')` |
| Regex extract | `regexp_matches()` | `REGEXP_EXTRACT()` |
| Nombre tabla | `schema.tabla` | `` `proyecto.dataset.tabla` `` |
| Valores nulos | `COALESCE()` | `COALESCE()` o `IFNULL()` |
| Partición | Índices | `PARTITION BY fecha` |
| Arrays | `ARRAY[1,2,3]` | `ARRAY[1,2,3]` o `['a','b','c']` |
| Tipos complejos | `JSON` | `STRUCT` |

---

## 10. Recursos para Aprender

### PostgreSQL
- Documentación oficial: https://www.postgresql.org/docs/
- Sitio de aprendizaje: https://pgexercises.com/
- Interactive SQL: https://sqlzoo.net/

### BigQuery
- Documentación: https://cloud.google.com/bigquery/docs
- SQL reference: https://cloud.google.com/bigquery/docs/reference/standard-sql
- Ejemplos públicos: `bigquery-public-data` proyecto
- Free tier: 1TB/mes de consultas

---

**El 90% de SQL es idéntico entre ambas. Las diferencias están en detalles de sintaxis y tipos de datos. ¡Aprende los fundamentos y adaptar será trivial!**
