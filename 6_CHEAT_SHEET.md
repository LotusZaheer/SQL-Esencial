# Referencia Rápida SQL - Cheat Sheet

## Tabla de Contenidos Rápida
- [SELECT & Filtros](#select--filtros)
- [Agregaciones](#agregaciones)
- [Window Functions](#window-functions)
- [JOINs](#joins)
- [Strings & Fechas](#strings--fechas)
- [CTEs & Subconsultas](#ctes--subconsultas)
- [Operadores y Funciones](#operadores-y-funciones)

---

## SELECT & Filtros

```sql
-- SELECT básico
SELECT columna1, columna2 FROM tabla;

-- WHERE: condiciones simples
WHERE edad > 25
WHERE edad = 30
WHERE edad != 30
WHERE estado IN ('activo', 'pendiente')
WHERE nombre LIKE 'Juan%'
WHERE email LIKE '%@gmail.com'
WHERE fecha BETWEEN '2024-01-01' AND '2024-12-31'
WHERE valor IS NULL
WHERE valor IS NOT NULL

-- Orden y límite
ORDER BY columna [ASC|DESC]
ORDER BY col1 DESC, col2 ASC
LIMIT 10
LIMIT 10 OFFSET 20  -- Página 3 (20 saltar, 10 resultar)

-- Valores únicos
SELECT DISTINCT ciudad FROM usuarios;
```

---

## Agregaciones

```sql
-- Funciones de agregación
COUNT(*)                -- Contar todas las filas
COUNT(columna)          -- Contar no-nulls
SUM(columna)            -- Suma
AVG(columna)            -- Promedio
MIN(columna)            -- Mínimo
MAX(columna)            -- Máximo
MAX(columna) FILTER (WHERE condicion)  -- Con filtro

-- Agrupar
GROUP BY columna1, columna2
HAVING COUNT(*) > 10    -- Filtrar grupos

-- Condicional con agregación
COUNT(CASE WHEN edad < 30 THEN 1 END) AS menores_30
SUM(CASE WHEN estado = 'activo' THEN monto ELSE 0 END)
```

---

## Window Functions

```sql
-- Ranking
ROW_NUMBER() OVER (ORDER BY columna)      -- 1,2,3,4...
RANK() OVER (ORDER BY columna)             -- 1,2,2,4...
DENSE_RANK() OVER (ORDER BY columna)       -- 1,2,2,3...

-- Primero y último
FIRST_VALUE(columna) OVER (ORDER BY ...)
LAST_VALUE(columna) OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)

-- Diferencia con filas anterior/posterior
LAG(columna) OVER (ORDER BY fecha)         -- Fila anterior
LEAD(columna) OVER (ORDER BY fecha)        -- Fila siguiente

-- Agregaciones en ventana
SUM(columna) OVER (ORDER BY fecha)         -- Suma acumulada
AVG(columna) OVER (PARTITION BY ciudad)    -- Promedio por ciudad

-- Percentiles
NTILE(4) OVER (ORDER BY ingresos)          -- Divide en 4 cuartiles
PERCENT_RANK() OVER (ORDER BY columna)     -- Percentil de cada fila
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY columna)  -- Mediana

-- Particionamiento
PARTITION BY ciudad       -- Ventanas por ciudad
ORDER BY fecha            -- Orden dentro de cada ventana
ROWS BETWEEN X AND Y      -- UNBOUNDED PRECEDING/FOLLOWING, CURRENT ROW, N PRECEDING/FOLLOWING
```

---

## JOINs

```sql
-- INNER JOIN (intersección)
FROM tabla_a a
INNER JOIN tabla_b b ON a.id = b.a_id

-- LEFT JOIN (todos del lado izquierdo)
FROM tabla_a a
LEFT JOIN tabla_b b ON a.id = b.a_id

-- RIGHT JOIN
FROM tabla_a a
RIGHT JOIN tabla_b b ON a.id = b.a_id

-- FULL OUTER JOIN (todo de ambos lados)
FROM tabla_a a
FULL OUTER JOIN tabla_b b ON a.id = b.a_id

-- CROSS JOIN (producto cartesiano)
FROM tabla_a
CROSS JOIN tabla_b

-- Self Join (tabla con ella misma)
FROM empleados e
LEFT JOIN empleados g ON e.gerente_id = g.id
```

---

## Strings & Fechas

### Funciones de String
```sql
-- Mayúsculas/Minúsculas
UPPER(columna)
LOWER(columna)
INITCAP(columna)        -- PostgreSQL

-- Largo
LENGTH(columna)

-- Substring
SUBSTR(columna, inicio, largo)
SUBSTRING(columna FROM inicio FOR largo)

-- Dividir
SPLIT_PART(columna, separador, posicion)  -- PostgreSQL
SPLIT_STRING(columna, separador)[OFFSET(n)]  -- BigQuery

-- Buscar posición
POSITION(substring IN columna)
STRPOS(columna, substring)  -- PostgreSQL

-- Reemplazar
REPLACE(columna, viejo, nuevo)

-- Espacios
TRIM(columna)           -- Ambos lados
LTRIM(columna)          -- Izquierda
RTRIM(columna)          -- Derecha

-- Concatenar
columna1 || columna2    -- PostgreSQL
CONCAT(columna1, columna2)
```

### Funciones de Fecha
```sql
-- Fecha/hora actual
CURRENT_DATE
CURRENT_TIMESTAMP
NOW()

-- Truncar fecha
DATE_TRUNC('month', fecha)     -- PostgreSQL
DATE_TRUNC(fecha, MONTH)       -- BigQuery

-- Extraer parte
EXTRACT(YEAR FROM fecha)
EXTRACT(MONTH FROM fecha)
EXTRACT(DAY FROM fecha)

-- Diferencia entre fechas
fecha1 - fecha2                 -- Días (PostgreSQL)
DATE_DIFF(fecha1, fecha2, DAY)  -- BigQuery
AGE(fecha1, fecha2)             -- PostgreSQL

-- Sumar/Restar días
fecha + INTERVAL '10 days'      -- PostgreSQL
DATE_ADD(fecha, INTERVAL 10 DAY)  -- BigQuery
DATEADD(day, 10, fecha)         -- SQL Server

-- Fecha a timestamp
CAST(fecha AS TIMESTAMP)
fecha::TIMESTAMP                -- PostgreSQL
```

---

## CTEs & Subconsultas

```sql
-- CTE simple
WITH usuarios_activos AS (
    SELECT id, nombre FROM usuarios WHERE estado = 'activo'
)
SELECT * FROM usuarios_activos;

-- CTEs múltiples
WITH
cte1 AS (SELECT ...),
cte2 AS (SELECT ... FROM cte1)
SELECT * FROM cte2;

-- Subconsulta en FROM
SELECT * FROM (
    SELECT id, nombre FROM usuarios WHERE estado = 'activo'
) filtered

-- Subconsulta en WHERE
SELECT * FROM usuarios
WHERE id IN (SELECT usuario_id FROM transacciones)

-- EXISTS / NOT EXISTS
SELECT * FROM usuarios u
WHERE EXISTS (SELECT 1 FROM transacciones WHERE usuario_id = u.id)
WHERE NOT EXISTS (SELECT 1 FROM transacciones WHERE usuario_id = u.id)
```

---

## Operadores y Funciones

### Operadores Lógicos
```sql
AND    -- Ambas condiciones verdaderas
OR     -- Al menos una condición verdadera
NOT    -- Negación

-- Combinación
WHERE (edad > 25 AND estado = 'activo')
   OR (ingresos > 100000)
```

### Condicionales
```sql
-- CASE simple
CASE WHEN condicion THEN valor ELSE otro_valor END

-- CASE múltiple
CASE
    WHEN edad < 18 THEN 'Menor'
    WHEN edad < 30 THEN 'Joven'
    ELSE 'Adulto'
END

-- COALESCE: primer valor no-null
COALESCE(columna1, columna2, 'valor_por_defecto')

-- NULLIF: retorna null si valores son iguales
NULLIF(columna, 0)

-- IFNULL (especialmente BigQuery)
IFNULL(columna, 0)
```

### Matemática
```sql
ROUND(numero, decimales)
CEIL(numero)
FLOOR(numero)
ABS(numero)
POWER(numero, exponente)
SQRT(numero)
MOD(numero, divisor)      -- Resto
GREATEST(num1, num2, ...)
LEAST(num1, num2, ...)
```

---

## Patrones Comunes

### Ranking dentro de grupo
```sql
SELECT
    nombre,
    ciudad,
    ingresos,
    ROW_NUMBER() OVER (PARTITION BY ciudad ORDER BY ingresos DESC) AS ranking
FROM usuarios
WHERE rn <= 3;  -- Top 3 por ciudad
```

### Suma acumulada
```sql
SELECT
    fecha,
    venta,
    SUM(venta) OVER (ORDER BY fecha) AS suma_acumulada,
    AVG(venta) OVER (ORDER BY fecha ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS promedio_7dias
FROM ventas;
```

### Cambio periodo a periodo
```sql
SELECT
    mes,
    ingresos,
    LAG(ingresos) OVER (ORDER BY mes) AS mes_anterior,
    ingresos - LAG(ingresos) OVER (ORDER BY mes) AS cambio,
    ROUND(((ingresos - LAG(ingresos) OVER (ORDER BY mes))
        / LAG(ingresos) OVER (ORDER BY mes) * 100), 2) AS cambio_pct
FROM ingresos_mensuales;
```

### Deduplicación
```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY usuario_id ORDER BY fecha DESC) AS rn
    FROM eventos
)
SELECT * FROM ranked WHERE rn = 1;
```

### Pivoting (filas a columnas)
```sql
SELECT
    usuario,
    SUM(CASE WHEN mes = 1 THEN ventas ELSE 0 END) AS enero,
    SUM(CASE WHEN mes = 2 THEN ventas ELSE 0 END) AS febrero,
    SUM(CASE WHEN mes = 3 THEN ventas ELSE 0 END) AS marzo
FROM ventas
GROUP BY usuario;
```

---

## Diferencias PostgreSQL vs BigQuery

| Operación | PostgreSQL | BigQuery |
|-----------|-----------|----------|
| Tabla | `schema.tabla` | `` `proyecto.dataset.tabla` `` |
| Truncar fecha | `DATE_TRUNC('month', col)` | `DATE_TRUNC(col, MONTH)` |
| Diferencia fecha | `fecha1 - fecha2` | `DATE_DIFF(fecha1, fecha2, DAY)` |
| Regex | `columna ~ 'pattern'` | `REGEXP_CONTAINS(columna, r'pattern')` |
| Array split | `SPLIT_PART()` | `SPLIT_STRING()[OFFSET(n)]` |
| Dividir string | `SPLIT_PART(col, ',', 1)` | `SPLIT_STRING(col, ',')[OFFSET(0)]` |
| Null si igual | `NULLIF()` | `NULLIF()` |
| Partición | Índices | `PARTITION BY` en CREATE TABLE |

---

## Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| "GROUP BY incomplete" | Columna en SELECT no en GROUP BY | Agregar a GROUP BY o agregar función |
| "NULL = NULL false" | NULL nunca es igual a NULL | Usar `IS NULL` en lugar de `= NULL` |
| "Implicit conversion overflow" | División entera | Usar `::NUMERIC` o `CAST()` |
| "Ambiguous reference" | Dos tablas con misma columna | Usar alias: `u.nombre` |
| "Division by zero" | Dividir entre cero | Usar `CASE WHEN denominador != 0` |

---

## Optimización Rápida

```sql
-- ✓ Bueno
SELECT u.nombre, COUNT(t.id)
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
WHERE u.estado = 'activo'
GROUP BY u.id, u.nombre;

-- ❌ Evitar
SELECT u.* FROM usuarios u
LEFT JOIN transacciones t USING (id);  -- Múltiples JOINs sin condición

SELECT *  -- BigQuery: escanea todas las columnas
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id;

SELECT COUNT(CASE WHEN (SELECT MAX(id) FROM...)...)  -- Subconsulta correlacionada
```

---

## Fórmulas Útiles

### Calcular edad
```sql
EXTRACT(YEAR FROM AGE(fecha_nacimiento))  -- PostgreSQL
DATE_DIFF(CURRENT_DATE(), fecha_nacimiento, YEAR)  -- BigQuery
DATEDIFF(YEAR, fecha_nacimiento, CURRENT_DATE)  -- SQL Server
```

### Porcentaje del total
```sql
ROUND(columna * 100.0 / SUM(columna) OVER (), 2)
```

### Quartiles/Percentiles
```sql
NTILE(4) OVER (ORDER BY columna)  -- Cuartiles
PERCENT_RANK() OVER (ORDER BY columna)
```

### Crecimiento porcentual
```sql
ROUND(((actual - anterior) / anterior * 100), 2)
```

### Filtro de valores nulos
```sql
COALESCE(columna, 0)  -- O usar IS NULL
```

---

**¡Imprime esta página o guárdala como favorito para referencias rápidas!**
