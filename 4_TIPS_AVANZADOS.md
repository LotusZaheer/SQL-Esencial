# Tips Avanzados, Mejores Prácticas y Troubleshooting

## 1. Performance: Optimización de Queries

### 1.1 - EXPLAIN: Lee el plan de ejecución

#### PostgreSQL
```sql
-- Ver cómo PostgreSQL ejecutará tu query
EXPLAIN ANALYZE
SELECT
    u.nombre,
    COUNT(t.id) as transacciones
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre;

-- Resultado típico:
-- Seq Scan on usuarios u  (cost=0.00..100.00)
-- Hash Left Join  (cost=50.00..150.00)
```

**Cosas a buscar:**
- `Seq Scan`: Escaneo secuencial (lento en tablas grandes)
- `Index Scan`: Usando índice (usualmente mejor)
- `cost`: número relativo de esfuerzo (menor = mejor)
- `Seq Scan` en gran tabla = problema (crea un índice)

#### BigQuery
```sql
-- BigQuery siempre muestra bytes escaneados
SELECT
    u.nombre,
    COUNT(t.id) as transacciones
FROM `proyecto.dataset.usuarios` u
LEFT JOIN `proyecto.dataset.transacciones` t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre;

-- En la UI verás:
-- Este trabajo escaneó 2.5 GB en 1.2 segundos
-- Te cuesta aproximadamente $0.01
```

---

### 1.2 - Crear Índices (PostgreSQL)

```sql
-- Índice simple (más usado)
CREATE INDEX idx_usuarios_estado ON usuarios(estado);

-- Índice compuesto (para queries con múltiples WHERE)
CREATE INDEX idx_usuarios_estado_ciudad ON usuarios(estado, ciudad);

-- Índice en columna calculada
CREATE INDEX idx_usuarios_email_lower ON usuarios(LOWER(email));

-- Índice BRIN (para series de tiempo)
CREATE INDEX idx_transacciones_fecha ON transacciones USING BRIN (fecha);

-- Ver índices
SELECT * FROM pg_indexes WHERE tablename = 'usuarios';

-- Eliminar índice no usado
DROP INDEX IF EXISTS idx_viejo;
```

**Cuándo crear índices:**
- Columnas en WHERE frecuentemente
- Columnas en JOINs (las del lado derecho especialmente)
- Columnas en ORDER BY de queries lentas
- NO crear índices en cada columna (ralentiza INSERTs y actualizations)

---

### 1.3 - Queries Lentas: Troubleshooting

#### PostgreSQL
```sql
-- Encontrar queries lentas
SELECT
    query,
    calls,
    total_time,
    mean_time,
    max_time
FROM pg_stat_statements
WHERE mean_time > 1000  -- queries que tardan más de 1 segundo promedio
ORDER BY total_time DESC;

-- Resetear estadísticas
SELECT pg_stat_statements_reset();

-- Ver qué está haciendo (en otra conexión)
SELECT pid, query, state FROM pg_stat_activity;
```

#### BigQuery
```sql
-- Jobs recientes y su costo
SELECT
    job_id,
    user_email,
    total_bytes_billed / POW(10,9) as gb_escanneados,
    total_slot_ms / 1000 as segundos_ejecucion,
    TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), creation_time, MINUTE) as hace_minutos
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= CURRENT_TIMESTAMP() - INTERVAL 24 HOUR
ORDER BY creation_time DESC
LIMIT 10;
```

---

### 1.4 - Evitar errores comunes

#### ❌ ANTI-PATTERN 1: JOIN sin filtrar
```sql
-- ❌ LENTO: Creates cartesian product si no hay filtro
SELECT * FROM usuarios u
CROSS JOIN ordenes o;  -- 1M usuarios x 10M ordenes = 10 billones filas!

-- ✓ CORRECTO: Agregar condición
SELECT u.nombre, o.numero
FROM usuarios u
INNER JOIN ordenes o ON u.id = o.usuario_id;
```

#### ❌ ANTI-PATTERN 2: Funciones en WHERE
```sql
-- ❌ LENTO: No puede usar índice
SELECT * FROM usuarios
WHERE LOWER(email) = 'test@gmail.com';  -- No usa índice en email

-- ✓ CORRECTO: Normalizar datos EN la tabla
SELECT * FROM usuarios
WHERE email = 'test@gmail.com';  -- Usa índice en email

-- O si necesitas búsqueda case-insensitive:
-- CREATE INDEX idx_usuarios_email_lower ON usuarios(LOWER(email));
SELECT * FROM usuarios
WHERE LOWER(email) = 'test@gmail.com';
```

#### ❌ ANTI-PATTERN 3: SELECT * con JOINs
```sql
-- ❌ LENTO en BigQuery: Escanea todas las columnas
SELECT * FROM usuarios u
JOIN ordenes o ON u.id = o.usuario_id;

-- ✓ CORRECTO: Solo las columnas necesarias
SELECT u.nombre, u.email, o.numero, o.monto
FROM usuarios u
JOIN ordenes o ON u.id = o.usuario_id;
```

#### ❌ ANTI-PATTERN 4: Subconsulta correlacionada
```sql
-- ❌ LENTO: La subconsulta se ejecuta para cada fila
SELECT
    u.nombre,
    (SELECT SUM(monto) FROM ordenes WHERE usuario_id = u.id) as total_ordenes
FROM usuarios u;

-- ✓ CORRECTO: Usar JOIN con GROUP BY
SELECT
    u.nombre,
    COALESCE(SUM(o.monto), 0) as total_ordenes
FROM usuarios u
LEFT JOIN ordenes o ON u.id = o.usuario_id
GROUP BY u.id, u.nombre;
```

---

## 2. Errores Comunes y Soluciones

### Error 1: "GROUP BY is incomplete"
```sql
-- ❌ ERROR: Columna no está en GROUP BY
SELECT
    nombre,
    email,
    COUNT(*) as registros
FROM usuarios
GROUP BY nombre;

-- ✓ CORRECTO: Agregar email a GROUP BY
SELECT
    nombre,
    email,
    COUNT(*) as registros
FROM usuarios
GROUP BY nombre, email;

-- O si solo quieres el nombre
SELECT
    nombre,
    COUNT(*) as registros
FROM usuarios
GROUP BY nombre;
```

### Error 2: "NULL en operaciones"
```sql
-- ❌ Comparar con NULL
SELECT * FROM usuarios WHERE ingresos = NULL;  -- Devuelve 0 filas!

-- ✓ CORRECTO: Usar IS NULL
SELECT * FROM usuarios WHERE ingresos IS NULL;

-- Para operaciones, usar COALESCE
SELECT
    nombre,
    COALESCE(ingresos, 0) + 1000 as ingresos_ajustados
FROM usuarios;
```

### Error 3: "DISTINCT no es lo que esperas"
```sql
-- ❌ Confusión: DISTINCT considera todas las columnas
SELECT DISTINCT ciudad, estado FROM usuarios;  -- Después usa parámetro incorrecto
SELECT DISTINCT ON (ciudad) ciudad, estado FROM usuarios;  -- PostgreSQL específico

-- ✓ CORRECTO en PostgreSQL:
SELECT DISTINCT ON (ciudad)
    ciudad,
    estado,
    nombre  -- muestra el primero de cada ciudad (sin ORDER BY es aleatorio)
FROM usuarios
ORDER BY ciudad, nombre;
```

### Error 4: "Integer division en SQL"
```sql
-- ❌ INCORRECTO: División entera
SELECT 5 / 2;  -- Resultado: 2 (en PostgreSQL sin casting)

-- ✓ CORRECTO: Cast a decimal
SELECT 5::NUMERIC / 2;  -- Resultado: 2.5
SELECT CAST(5 AS NUMERIC) / 2;  -- Resultado: 2.5

-- O para porcentajes:
SELECT ROUND((total_pagado * 100.0 / total_debido), 2) AS porcentaje;  -- .0 fuerza float
```

### Error 5: "Transacciones no se commitean"
```sql
-- ❌ Abrir transacción y no cerrar
BEGIN;
UPDATE usuarios SET estado = 'activo' WHERE edad > 18;
-- Cambios quedan en transacción abierta, otros no los ven

-- ✓ CORRECTO: COMMIT o ROLLBACK
BEGIN;
UPDATE usuarios SET estado = 'activo' WHERE edad > 18;
COMMIT;  -- Confirmar cambios

-- O rollback si cambias de idea:
BEGIN;
DELETE FROM transacciones WHERE antigua = true;
ROLLBACK;  -- Cancela el DELETE
```

---

## 3. Patrones Útiles

### Patrón 1: Deduplicación
```sql
-- Obtener el registro más nuevo de cada usuario
WITH ranked AS (
    SELECT
        usuario_id,
        fecha,
        datos,
        ROW_NUMBER() OVER (PARTITION BY usuario_id ORDER BY fecha DESC) as rn
    FROM eventos
)
SELECT *
FROM ranked
WHERE rn = 1;

-- Alternativa más simple:
SELECT DISTINCT ON (usuario_id)
    usuario_id,
    fecha,
    datos
FROM eventos
ORDER BY usuario_id, fecha DESC;  -- PostgreSQL DISTINCT ON
```

### Patrón 2: Pivoting (convertir filas a columnas)
```sql
-- Antes:
-- usuario | mes | ventas
-- A       | 1   | 1000
-- A       | 2   | 1500
-- B       | 1   | 800

-- Después: columnas por mes
SELECT
    usuario,
    SUM(CASE WHEN mes = 1 THEN ventas ELSE 0 END) as mes_1,
    SUM(CASE WHEN mes = 2 THEN ventas ELSE 0 END) as mes_2,
    SUM(CASE WHEN mes = 3 THEN ventas ELSE 0 END) as mes_3
FROM ventas
GROUP BY usuario;

-- Resultado:
-- usuario | mes_1 | mes_2 | mes_3
-- A       | 1000  | 1500  | NULL
-- B       | 800   | NULL  | NULL
```

### Patrón 3: Unpivoting (convertir columnas a filas)
```sql
-- PostgreSQL: jsonb_each o UNION
SELECT usuario, 'mes_1' as mes, mes_1 as ventas FROM ventas_pivot
UNION ALL
SELECT usuario, 'mes_2' as mes, mes_2 as ventas FROM ventas_pivot
UNION ALL
SELECT usuario, 'mes_3' as mes, mes_3 as ventas FROM ventas_pivot;

-- BigQuery: UNNEST con arrays
SELECT
    usuario,
    mes,
    ventas
FROM `proyecto.dataset.ventas`,
UNNEST(['mes_1', 'mes_2', 'mes_3']) as mes
WITH OFFSET as idx
WHERE ventas IS NOT NULL;
```

### Patrón 4: Running Totals (sumas acumuladas)
```sql
SELECT
    fecha,
    venta,
    SUM(venta) OVER (ORDER BY fecha) as suma_acumulada,
    SUM(venta) OVER (
        ORDER BY fecha
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
    ) as suma_anterior,
    venta - LAG(venta) OVER (ORDER BY fecha) as diferencia
FROM ventas_diarias
ORDER BY fecha;
```

### Patrón 5: Percent to Total
```sql
SELECT
    categoria,
    nombre_producto,
    ventas,
    SUM(ventas) OVER (PARTITION BY categoria) as total_categoria,
    ROUND(ventas * 100.0 / SUM(ventas) OVER (PARTITION BY categoria), 2) as porcentaje
FROM productos
ORDER BY categoria, porcentaje DESC;
```

---

## 4. Mejores Prácticas de Código SQL

### Estilo de Código
```sql
-- ✓ BUENO: Legible
SELECT
    u.id,
    u.nombre,
    COUNT(DISTINCT t.id) AS transacciones,
    ROUND(SUM(t.monto), 2) AS total
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
WHERE u.estado = 'activo'
GROUP BY u.id, u.nombre
HAVING COUNT(DISTINCT t.id) > 0
ORDER BY total DESC;

-- ❌ MALO: Ilegible
SELECT u.id,u.nombre,COUNT(DISTINCT t.id)AS transacciones,ROUND(SUM(t.monto),2)AS total FROM usuarios u LEFT JOIN transacciones t ON u.id=t.usuario_id WHERE u.estado='activo'GROUP BY u.id,u.nombre HAVING COUNT(DISTINCT t.id)>0 ORDER BY total DESC;
```

### Comentarios
```sql
-- ✓ COMENTARIO ÚTIL
-- Obtener usuarios activos que hayan tenido transacciones en el último mes
SELECT
    u.nombre
FROM usuarios u
WHERE u.estado = 'activo'
  AND EXISTS (
      SELECT 1 FROM transacciones t
      WHERE t.usuario_id = u.id
      AND t.fecha >= CURRENT_DATE - INTERVAL '1 month'
  );

-- ❌ COMENTARIO INÚTIL
-- Seleccionar nombre de usuarios
SELECT u.nombre FROM usuarios u;  -- obvio del código
```

### Alias
```sql
-- ✓ BUENOS ALIAS
SELECT
    u.id AS usuario_id,
    u.nombre AS usuario_nombre,
    d.nombre AS departamento_nombre
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id;

-- ❌ ALIAS CONFUSOS
SELECT
    u.id AS id1,
    u.nombre AS col2,
    d.nombre AS d
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id;
```

---

## 5. Debugging: Cómo Resolver Queries Rotas

### Paso 1: Simplificar
```sql
-- Si esta query no funciona:
SELECT u.nombre, COUNT(t.id), AVG(t.monto)
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
LEFT JOIN productos p ON t.producto_id = p.id
WHERE u.estado = 'activo' AND p.categoria IN ('A', 'B')
GROUP BY u.id
HAVING COUNT(t.id) > 10
ORDER BY COUNT(t.id) DESC
LIMIT 10;

-- Simplificar paso a paso:
-- Paso 1: Solo tabla usuarios
SELECT * FROM usuarios WHERE u.estado = 'activo';

-- Paso 2: Agregar primer JOIN
SELECT u.nombre, COUNT(t.id)
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre;

-- Paso 3: Agregar SELECT del segundo JOIN
SELECT u.nombre, p.categoria, COUNT(t.id)
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
LEFT JOIN productos p ON t.producto_id = p.id
GROUP BY u.id, u.nombre, p.categoria;

-- Paso 4: Agregar WHERE
-- ... y así sucesivamente
```

### Paso 2: Verificar tipos de datos
```sql
-- ¿Está comparando manzanas con naranjas?
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'usuarios';

-- Si joins no funcionan, tipos incompatibles:
SELECT u.id::INTEGER, t.usuario_id::INTEGER
FROM usuarios u
JOIN transacciones t ON u.id = t.usuario_id;
```

### Paso 3: Contar registros
```sql
-- Verificar que tienes datos
SELECT COUNT(*) FROM usuarios;  -- 1000 registros
SELECT COUNT(*) FROM transacciones;  -- 50000 registros

-- Después de join, ¿cuántos quedan?
SELECT COUNT(*) FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id;

-- ¿Muchos más? Hay duplicados. Usar DISTINCT ON / ROW_NUMBER
```

---

## 6. Testing y Documentación

### Crear Test Cases
```sql
-- Crear usuarios de prueba
INSERT INTO usuarios (nombre, email, estado) VALUES
    ('Test User 1', 'test1@test.com', 'activo'),
    ('Test User 2', 'test2@test.com', 'inactivo'),
    ('Test User 3', 'test3@test.com', 'activo');

-- Tu query debe retornar solo usuarios activos y con transacciones
SELECT COUNT(*) as resultado_esperado FROM usuarios
WHERE estado = 'activo' AND EXISTS (
    SELECT 1 FROM transacciones WHERE usuario_id = usuarios.id
);
-- Resultado esperado: 2
```

### Documentar queries complejas
```sql
-- Query: Análisis de RFM (Recency, Frequency, Monetary)
-- Propósito: Segmentar clientes para estrategia de marketing
-- Autор: Tu Nombre
-- Fecha: 2024-02-09
-- Última modificación: agregar percentiles
-- Dependencias: tabla_transacciones, tabla_usuarios
-- Costo estimado (BigQuery): 500MB
-- Tiempo estimado de ejecución: 2-3 segundos

WITH ...
```

---

## 7. Links Útiles para Aprender Más

### PostgreSQL
- **Documentación oficial**: https://www.postgresql.org/docs/current/
- **PgExercises** (práctica): https://pgexercises.com/
- **Use The Index, Luke** (índices): https://use-the-index-luke.com/
- **PostGIS** (para datos geográficos): https://postgis.net/

### BigQuery
- **Documentación**: https://cloud.google.com/bigquery/docs
- **SQL Reference**: https://cloud.google.com/bigquery/docs/reference/standard-sql
- **Public Datasets**: `bigquery-public-data` project
- **Cost Calculator**: https://cloud.google.com/products/calculator

### General SQL
- **SqlZoo**: https://sqlzoo.net/ (interactivo)
- **LeetCode Database**: https://leetcode.com/problemset/database/
- **HackerRank SQL**: https://www.hackerrank.com/domains/sql
- **Mode Analytics SQL Tutorial**: https://mode.com/sql-tutorial/

---

**¡Ahora estás listo para escribir SQL profesional y eficiente!**
