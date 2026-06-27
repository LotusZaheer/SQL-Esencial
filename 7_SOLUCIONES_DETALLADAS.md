# Soluciones Detalladas de Ejercicios

Este documento contiene las respuestas a los ejercicios del fichero `2_EJERCICIOS_PRACTICOS.md` con explicaciones paso a paso.

---

## NIVEL 1: Respuestas

### 1.1 - SELECT y WHERE

**Ejercicio:** Obtener nombre, email y edad de todos los usuarios activos ordenados por edad descendente.

**Solución:**
```sql
SELECT
    nombre,
    email,
    edad
FROM usuarios
WHERE estado = 'activo'
ORDER BY edad DESC;
```

**Explicación paso a paso:**
1. `SELECT nombre, email, edad` → Elige las 3 columnas que necesitamos
2. `FROM usuarios` → De la tabla usuarios
3. `WHERE estado = 'activo'` → Solo filtra filas donde estado es activo (descarta otros estados)
4. `ORDER BY edad DESC` → Ordena por edad de mayor a menor (descendente)

**Resultado esperado:**
```
nombre    | email         | edad
----------|---------------|------
Juan      | juan@mail.com | 45
María     | maria@mail.com| 38
Pedro     | pedro@mail.com| 32
...
```

---

### 1.2 - Agregación básica

**Ejercicio:** Contar cuántos usuarios hay en total y calcular la edad promedio.

**Solución:**
```sql
SELECT
    COUNT(*) AS total_usuarios,
    ROUND(AVG(edad), 2) AS edad_promedio,
    MIN(edad) AS edad_minima,
    MAX(edad) AS edad_maxima
FROM usuarios;
```

**Explicación:**
- `COUNT(*)` cuenta todas las filas (todos los usuarios)
- `AVG(edad)` calcula el promedio de edades
- `ROUND(..., 2)` redondea a 2 decimales (si edad_promedio es 34.56789, devuelve 34.57)
- `MIN(edad)` y `MAX(edad)` encuentran los extremos
- No hay GROUP BY porque queremos UN resultado con estadísticas globales

**Resultado esperado:**
```
total_usuarios | edad_promedio | edad_minima | edad_maxima
1000           | 35.47         | 18          | 75
```

---

### 1.3 - GROUP BY simple

**Ejercicio:** Contar cuántos usuarios hay en cada ciudad.

**Solución:**
```sql
SELECT
    ciudad,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
ORDER BY cantidad_usuarios DESC;
```

**Explicación paso a paso:**
1. `SELECT ciudad, COUNT(*)` → Queremos ver la ciudad y cuántos usuarios hay
2. `GROUP BY ciudad` → Agrupar por ciudad (todos los usuarios de Madrid juntos, Barcelona juntos, etc.)
3. `COUNT(*)` cuenta cuántos hay en cada grupo
4. `ORDER BY cantidad_usuarios DESC` → Muestra la ciudad con más usuarios primero

**Resultado esperado:**
```
ciudad      | cantidad_usuarios
Madrid      | 350
Barcelona   | 280
Valencia    | 190
...
```

---

## NIVEL 2: Respuestas

### 2.1 - GROUP BY con HAVING

**Ejercicio:** Mostrar las ciudades que tienen más de 10 usuarios, con su edad promedio e ingresos totales.

**Solución:**
```sql
SELECT
    ciudad,
    COUNT(*) AS cantidad_usuarios,
    ROUND(AVG(edad), 1) AS edad_promedio,
    ROUND(SUM(ingresos), 2) AS ingresos_totales
FROM usuarios
WHERE estado = 'activo'
GROUP BY ciudad
HAVING COUNT(*) > 10
ORDER BY cantidad_usuarios DESC;
```

**Explicación:**
- `WHERE estado = 'activo'` → Se ejecuta PRIMERO, filtra solo usuarios activos
- `GROUP BY ciudad` → Agrupa los usuarios activos por ciudad
- `HAVING COUNT(*) > 10` → Se ejecuta DESPUÉS de agrupar, mantiene solo ciudades con >10 usuarios
- Diferencia clave:
  - WHERE: filtra filas (antes de group by)
  - HAVING: filtra grupos (después de group by)

**Comparación:**
```sql
-- ❌ INCORRECTO: COUNT(*) no es una columna, no puedes usarlo en WHERE
WHERE COUNT(*) > 10  -- ERROR

-- ✓ CORRECTO: HAVING para filtrar después de group by
HAVING COUNT(*) > 10
```

---

### 2.2 - CASE condicional

**Ejercicio:** Categorizar usuarios por ingresos: Bajo (<30k), Medio (30k-70k), Alto (>70k).

**Solución:**
```sql
SELECT
    nombre,
    ingresos,
    CASE
        WHEN ingresos < 30000 THEN 'Bajo'
        WHEN ingresos <= 70000 THEN 'Medio'
        ELSE 'Alto'
    END AS categoria_ingreso
FROM usuarios
ORDER BY ingresos DESC;
```

**Explicación del CASE:**
```
CASE
    WHEN ingresos < 30000 THEN 'Bajo'           ← Si ingreso < 30000, retorna 'Bajo'
    WHEN ingresos <= 70000 THEN 'Medio'         ← Si NO se cumplió antes, pero <= 70000, retorna 'Medio'
    ELSE 'Alto'                                  ← Si no se cumplió ninguna condición anterior, retorna 'Alto'
END
```

**Traza ejemplo:**
```
Usuario A: 25000
→ ¿25000 < 30000? Sí → return 'Bajo' (fin)

Usuario B: 50000
→ ¿50000 < 30000? No
→ ¿50000 <= 70000? Sí → return 'Medio' (fin)

Usuario C: 100000
→ ¿100000 < 30000? No
→ ¿100000 <= 70000? No
→ return 'Alto'
```

---

### 2.3 - Múltiples agregaciones con CASE

**Ejercicio:** Mostrar por ciudad: cantidad total de usuarios, cuántos activos, cuántos inactivos, ingresos promedio.

**Solución:**
```sql
SELECT
    ciudad,
    COUNT(*) AS total_usuarios,
    COUNT(CASE WHEN estado = 'activo' THEN 1 END) AS usuarios_activos,
    COUNT(CASE WHEN estado = 'inactivo' THEN 1 END) AS usuarios_inactivos,
    COUNT(CASE WHEN estado = 'suspendido' THEN 1 END) AS usuarios_suspendidos,
    ROUND(AVG(ingresos), 2) AS ingresos_promedio
FROM usuarios
GROUP BY ciudad
ORDER BY total_usuarios DESC;
```

**Explicación del truco `COUNT(CASE WHEN...)`:**
```sql
COUNT(CASE WHEN estado = 'activo' THEN 1 END)
```

¿Qué sucede aquí?
- Si `estado = 'activo'`, retorna `1`
- Si no, retorna `NULL` (implícitamente)
- `COUNT()` cuenta solo valores no-NULL
- Resultado: cuenta cuántos usuarios son activos

**Alternativa (más verbose):**
```sql
SUM(CASE WHEN estado = 'activo' THEN 1 ELSE 0 END)  -- También funciona
```

---

### 2.4 - Fecha y operaciones

**Ejercicio:** Mostrar nombre, email, y cuántos días llevan registrados los usuarios.

**Solución:**
```sql
SELECT
    nombre,
    email,
    fecha_registro,
    CURRENT_DATE - fecha_registro AS dias_registrado,
    ROUND((CURRENT_DATE - fecha_registro) / 365.25, 1) AS años_registrado
FROM usuarios
ORDER BY fecha_registro DESC;
```

**Explicación:**
- `CURRENT_DATE` obtiene la fecha de hoy
- `CURRENT_DATE - fecha_registro` = número de días (en PostgreSQL)
- Dividir por 365.25 (año con decimales) = años aproximados

**Ejemplo:**
```
Si hoy es 2024-02-09 y fecha_registro es 2023-02-09:
CURRENT_DATE - fecha_registro = 365 días
365 / 365.25 ≈ 1.0 años
```

---

## NIVEL 3: Respuestas

### 3.1 - INNER JOIN simple

**Ejercicio:** Mostrar nombre de usuario, departamento y salario.

**Solución:**
```sql
SELECT
    u.nombre,
    d.nombre AS departamento,
    u.ingresos
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id;
```

**Explicación del INNER JOIN:**
```
Tabla usuarios:          Tabla departamentos:
id | nombre | depto_id  id | nombre
1  | Juan   | 10        10 | Ventas
2  | María  | 20        20 | IT
3  | Pedro  | NULL      30 | RRHH

INNER JOIN:
id | nombre | depto_id | depto_id | depto_nombre
1  | Juan   | 10       | 10       | Ventas
2  | María  | 20       | 20       | IT
(Pedro se descarta porque no existe depto_id NULL en departamentos)
```

**Diferencia INNER vs LEFT:**
- `INNER JOIN`: solo registros con match en AMBAS tablas
- `LEFT JOIN`: todos del lado izquierdo, aunque no haya match

---

### 3.2 - LEFT JOIN con COALESCE

**Ejercicio:** Mostrar todos los usuarios con sus departamentos, aunque no tengan.

**Solución:**
```sql
SELECT
    u.id,
    u.nombre,
    COALESCE(d.nombre, 'Sin asignar') AS departamento,
    u.ingresos
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
ORDER BY u.nombre;
```

**Por qué COALESCE?**
```sql
-- Sin COALESCE:
SELECT u.nombre, d.nombre AS departamento
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id;

-- Resultado:
nombre | departamento
Juan   | Ventas
María  | IT
Pedro  | NULL        ← Incómodo sin mostrar algo significativo

-- Con COALESCE:
COALESCE(d.nombre, 'Sin asignar')

-- Resultado:
nombre | departamento
Juan   | Ventas
María  | IT
Pedro  | Sin asignar  ← Mejor legibilidad
```

---

### 3.3 - Múltiples JOINs con agregación

**Ejercicio:** Usuarios, departamentos, cantidad de proyectos e horas totales.

**Solución:**
```sql
SELECT
    u.nombre AS usuario,
    d.nombre AS departamento,
    COUNT(DISTINCT a.proyecto_id) AS cantidad_proyectos,
    ROUND(SUM(a.horas), 2) AS horas_totales
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
LEFT JOIN asignaciones a ON u.id = a.usuario_id
WHERE u.estado = 'activo'
GROUP BY u.id, u.nombre, d.id, d.nombre
ORDER BY horas_totales DESC NULLS LAST;
```

**Traza paso a paso:**

1. **Tabla resultado después de 1er JOIN:**
```
u.id | u.nombre | u.estado | d.id | d.nombre
1    | Juan     | activo   | 10   | Ventas
2    | María    | activo   | 20   | IT
3    | Pedro    | inactivo | NULL | NULL
```

2. **Después de 2do JOIN (asignaciones):**
```
u.id | u.nombre | d.nombre | a.proyecto_id | a.horas
1    | Juan     | Ventas   | 101           | 40
1    | Juan     | Ventas   | 102           | 30
2    | María    | IT       | 103           | 50
3    | Pedro    | NULL     | NULL          | NULL
```

3. **Después de WHERE (estado = 'activo'):**
```
u.id | u.nombre | d.nombre | a.proyecto_id | a.horas
1    | Juan     | Ventas   | 101           | 40
1    | Juan     | Ventas   | 102           | 30
2    | María    | IT       | 103           | 50
```

4. **Después de GROUP BY:**
```
u.nombre | d.nombre | COUNT(DISTINCT...) | SUM(horas)
Juan     | Ventas   | 2                  | 70
María    | IT       | 1                  | 50
```

---

## NIVEL 4: Respuestas

### 4.1 - Window Functions: ROW_NUMBER

**Ejercicio:** Numerar usuarios por ingresos dentro de cada ciudad.

**Solución:**
```sql
SELECT
    nombre,
    ciudad,
    ingresos,
    ROW_NUMBER() OVER (PARTITION BY ciudad ORDER BY ingresos DESC) AS ranking_en_ciudad,
    RANK() OVER (PARTITION BY ciudad ORDER BY ingresos DESC) AS rango,
    DENSE_RANK() OVER (PARTITION BY ciudad ORDER BY ingresos DESC) AS rango_denso
FROM usuarios
WHERE estado = 'activo'
ORDER BY ciudad, ranking_en_ciudad;
```

**Diferencia entre ROW_NUMBER, RANK, DENSE_RANK:**

```
Datos de ejemplo (ciudad = Madrid):
nombre | ingresos | ROW_NUMBER | RANK | DENSE_RANK
Ana    | 80000    | 1          | 1    | 1
Bob    | 80000    | 2          | 1    | 1       ← Empate
Carlos | 70000    | 3          | 3    | 2       ← RANK salta números
Diego  | 70000    | 4          | 3    | 2
Eva    | 60000    | 5          | 5    | 3       ← RANK salta a 5

↑          ↑          ↑          ↑
Nombre  Ingresos   Lidera 1,2,3 Lidera 1,1,3 Lidera 1,1,2

ROW_NUMBER:   Siempre única (1,2,3,4,5...)
RANK:         Salta si hay empate (1,1,3,3,5...)
DENSE_RANK:   No salta (1,1,2,2,3...)
```

---

### 4.2 - Suma acumulada

**Ejercicio:** Transacciones por usuario con suma acumulada.

**Solución:**
```sql
SELECT
    u.nombre,
    t.fecha,
    t.monto,
    SUM(t.monto) OVER (
        PARTITION BY u.id
        ORDER BY t.fecha
    ) AS suma_acumulada,
    SUM(t.monto) OVER (PARTITION BY u.id) AS total_usuario
FROM transacciones t
JOIN usuarios u ON t.usuario_id = u.id
WHERE EXTRACT(YEAR FROM t.fecha) = EXTRACT(YEAR FROM CURRENT_DATE)
ORDER BY u.id, t.fecha;
```

**Cómo funciona la suma acumulada:**
```
nombre | fecha      | monto | suma_acumulada | total_usuario
Juan   | 2024-01-01 | 100   | 100            | 500
Juan   | 2024-01-15 | 150   | 250            | 500     ← 100 + 150
Juan   | 2024-02-01 | 200   | 450            | 500     ← 100 + 150 + 200
Juan   | 2024-03-01 | 50    | 500            | 500     ← suma de todas

total_usuario no suma = la SUMA TOTAL sin importar fecha (sin ORDER BY)
suma_acumulada suma = acumulado HASTA esa fila (con ORDER BY fecha)
```

---

### 4.3 - LAG para comparación período a período

**Ejercicio:** Cambio de ingresos mes a mes.

**Solución:**
```sql
SELECT
    nombre,
    mes,
    ingresos_mes,
    LAG(ingresos_mes) OVER (PARTITION BY user_id ORDER BY mes) AS mes_anterior,
    ingresos_mes - LAG(ingresos_mes) OVER (PARTITION BY user_id ORDER BY mes) AS cambio_absoluto,
    ROUND(
        ((ingresos_mes - LAG(ingresos_mes) OVER (PARTITION BY user_id ORDER BY mes))
        / LAG(ingresos_mes) OVER (PARTITION BY user_id ORDER BY mes) * 100),
        2
    ) AS cambio_porcentaje
FROM usuarios_ingresos_mensuales
ORDER BY user_id, mes;
```

**Cómo funciona LAG:**
```
mes    | ingresos_mes | LAG(ingresos_mes) | cambio
2024-01| 1000         | NULL              | NULL      ← Sin mes anterior
2024-02| 1100         | 1000              | 100       ← 1100 - 1000
2024-03| 1050         | 1100              | -50       ← 1050 - 1100
2024-04| 1200         | 1050              | 150       ← 1200 - 1050

cambio_porcentaje para 2024-02:
((1100 - 1000) / 1000 * 100) = (100 / 1000 * 100) = 10%
```

---

## NIVEL 5: Respuestas

### 5.1 - CTE simple

**Ejercicio:** Usuarios activos con ingresos > promedio.

**Solución:**
```sql
WITH usuarios_activos AS (
    SELECT id, nombre, ingresos
    FROM usuarios
    WHERE estado = 'activo'
),
promedio_ingresos AS (
    SELECT AVG(ingresos) AS promedio
    FROM usuarios_activos
)
SELECT
    ua.nombre,
    ua.ingresos,
    pi.promedio,
    ua.ingresos - pi.promedio AS diferencia
FROM usuarios_activos ua
CROSS JOIN promedio_ingresos pi
WHERE ua.ingresos > pi.promedio
ORDER BY ua.ingresos DESC;
```

**Explicación:**
1. Primera CTE: filtra usuarios activos
2. Segunda CTE: calcula el promedio de los activos
3. CROSS JOIN con tabla de 1 fila: permite usar la constante promedio en WHERE

**Alternativa más simple (sin CROSS JOIN):**
```sql
WITH usuarios_activos AS (
    SELECT id, nombre, ingresos
    FROM usuarios
    WHERE estado = 'activo'
)
SELECT
    nombre,
    ingresos,
    (SELECT AVG(ingresos) FROM usuarios_activos) AS promedio,
    ingresos - (SELECT AVG(ingresos) FROM usuarios_activos) AS diferencia
FROM usuarios_activos
WHERE ingresos > (SELECT AVG(ingresos) FROM usuarios_activos)
ORDER BY ingresos DESC;
```

---

## NIVEL 6: Ejercicios Complejos

### 6.1 - EXISTS / NOT EXISTS

**Ejercicio:** Usuarios con actividad el año pasado pero no este año.

**Solución:**
```sql
SELECT
    u.id,
    u.nombre,
    u.email
FROM usuarios u
WHERE EXISTS (
    SELECT 1
    FROM transacciones t
    WHERE t.usuario_id = u.id
    AND EXTRACT(YEAR FROM t.fecha) = EXTRACT(YEAR FROM CURRENT_DATE) - 1
)
AND NOT EXISTS (
    SELECT 1
    FROM transacciones t
    WHERE t.usuario_id = u.id
    AND EXTRACT(YEAR FROM t.fecha) = EXTRACT(YEAR FROM CURRENT_DATE)
)
ORDER BY u.nombre;
```

**Qué significa EXISTS:**
```sql
EXISTS (subquery)  → Verdadero si la subquery retorna AL MENOS una fila
NOT EXISTS         → Verdadero si la subquery NO retorna ninguna fila

En este caso:
EXISTS (...año pasado...)      → El usuario tuvo transacciones el año pasado
NOT EXISTS (...este año...)    → El usuario NO tuvo transacciones este año
```

---

## Ejercicios Desafío - Soluciones

### Desafío 1: Crecimiento Mes a Mes

**Solución propuesta:**
```sql
WITH transacciones_mensuales AS (
    SELECT
        DATE_TRUNC('month', fecha)::date AS mes,
        COUNT(*) AS num_transacciones,
        SUM(monto) AS total_mes
    FROM transacciones
    GROUP BY DATE_TRUNC('month', fecha)
)
SELECT
    mes,
    num_transacciones,
    total_mes,
    LAG(num_transacciones) OVER (ORDER BY mes) AS trans_mes_anterior,
    ROUND(
        ((num_transacciones - LAG(num_transacciones) OVER (ORDER BY mes))
        / LAG(num_transacciones) OVER (ORDER BY mes) * 100),
        2
    ) AS crecimiento_porcentaje
FROM transacciones_mensuales
ORDER BY mes DESC;
```

---

### Desafío 2: Top 3 Productos por Categoría

**Solución propuesta:**
```sql
WITH ventas_ranking AS (
    SELECT
        p.categoria,
        p.nombre AS producto,
        SUM(v.cantidad) AS cantidad_vendida,
        SUM(v.monto) AS ingresos_totales,
        ROW_NUMBER() OVER (
            PARTITION BY p.categoria
            ORDER BY SUM(v.cantidad) DESC
        ) AS ranking_en_categoria
    FROM ventas v
    JOIN productos p ON v.producto_id = p.id
    GROUP BY p.categoria, p.nombre
)
SELECT *
FROM ventas_ranking
WHERE ranking_en_categoria <= 3
ORDER BY categoria, ranking_en_categoria;
```

---

**¡Felicidades por completar los ejercicios!** 🎉

El siguiente paso es practicar con tu propia base de datos y crear queries reales.
