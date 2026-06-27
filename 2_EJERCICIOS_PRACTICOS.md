# Ejercicios Prácticos de SQL

## Base de Datos de Ejemplo

Para todos estos ejercicios, asumimos estas tablas:

```sql
-- Tabla de usuarios
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    apellido VARCHAR(100),
    email VARCHAR(100),
    edad INT,
    ciudad VARCHAR(50),
    estado VARCHAR(20),  -- activo, inactivo, suspendido
    ingresos DECIMAL(10,2),
    fecha_registro DATE,
    fecha_nacimiento DATE
);

-- Tabla de departamentos
CREATE TABLE departamentos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    ciudad VARCHAR(50),
    presupuesto DECIMAL(12,2)
);

-- Tabla de asignaciones (usuarios a proyectos)
CREATE TABLE asignaciones (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES usuarios(id),
    proyecto_id INT,
    horas DECIMAL(5,2),
    estado VARCHAR(20)
);

-- Tabla de transacciones
CREATE TABLE transacciones (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES usuarios(id),
    monto DECIMAL(10,2),
    fecha TIMESTAMP,
    tipo VARCHAR(20),
    descripcion VARCHAR(255)
);

-- Tabla de proyectos
CREATE TABLE proyectos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    departamento_id INT REFERENCES departamentos(id),
    fecha_inicio DATE,
    fecha_fin DATE,
    presupuesto DECIMAL(12,2)
);
```

---

## NIVEL 1: Ejercicios Básicos

### 1.1 - SELECT y WHERE
**Ejercicio:** Obtener nombre, email y edad de todos los usuarios activos ordenados por edad descendente.

**Dificultad:** ⭐ Muy Fácil

<details>
<summary>Ver Solución</summary>

```sql
SELECT
    nombre,
    email,
    edad
FROM usuarios
WHERE estado = 'activo'
ORDER BY edad DESC;
```

**Explicación:**
- `SELECT` para elegir las 3 columnas
- `WHERE estado = 'activo'` filtra solo activos
- `ORDER BY edad DESC` ordena de mayor a menor edad
</details>

---

### 1.2 - Agregación básica
**Ejercicio:** Contar cuántos usuarios hay en total y calcular la edad promedio.

**Dificultad:** ⭐ Muy Fácil

<details>
<summary>Ver Solución</summary>

```sql
SELECT
    COUNT(*) AS total_usuarios,
    ROUND(AVG(edad), 2) AS edad_promedio,
    MIN(edad) AS edad_minima,
    MAX(edad) AS edad_maxima
FROM usuarios;
```

**Explicación:**
- `COUNT(*)` cuenta todas las filas
- `AVG(edad)` calcula el promedio
- `MIN/MAX` encuentra extremos
- `ROUND(..., 2)` redondea a 2 decimales
</details>

---

### 1.3 - GROUP BY simple
**Ejercicio:** Contar cuántos usuarios hay en cada ciudad.

**Dificultad:** ⭐ Muy Fácil

<details>
<summary>Ver Solución</summary>

```sql
SELECT
    ciudad,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
ORDER BY cantidad_usuarios DESC;
```

**Explicación:**
- `GROUP BY ciudad` agrupa por ciudad
- `COUNT(*)` cuenta los usuarios en cada grupo
- `ORDER BY cantidad DESC` muestra las ciudades con más usuarios primero
</details>

---

## NIVEL 2: Ejercicios Intermedios

### 2.1 - GROUP BY con HAVING
**Ejercicio:** Mostrar las ciudades que tienen más de 10 usuarios, con su edad promedio e ingresos totales.

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

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
- `WHERE estado = 'activo'` se ejecuta primero (antes de agrupar)
- `GROUP BY ciudad` agrupa los datos
- `HAVING COUNT(*) > 10` filtra los grupos (después de agrupar)
- La diferencia: WHERE es antes de agrupar, HAVING es después
</details>

---

### 2.2 - CASE condicional
**Ejercicio:** Categorizar usuarios por ingresos: Bajo (<30k), Medio (30k-70k), Alto (>70k).

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `CASE WHEN ... THEN ... ELSE ... END` evalúa condiciones
- Se evalúan en orden, la primera verdadera gana
- El último `ELSE` es para valores que no coinciden
</details>

---

### 2.3 - Múltiples agregaciones con CASE
**Ejercicio:** Mostrar por ciudad: cantidad total de usuarios, cuántos activos, cuántos inactivos, ingresos promedio.

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `COUNT(CASE WHEN...)` es muy poderoso: cuenta solo los que cumplen la condición
- Mucho más eficiente que filtrar con WHERE para cada categoría
</details>

---

### 2.4 - Fecha y operaciones
**Ejercicio:** Mostrar nombre, email, y cuántos días llevan registrados los usuarios, ordenado por más recientes.

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

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
- `CURRENT_DATE` obtiene la fecha actual
- Restar fechas en PostgreSQL da el número de días
- Dividir por 365.25 convierte a años aproximados
</details>

---

## NIVEL 3: Ejercicios con JOINs

### 3.1 - INNER JOIN simple
**Ejercicio:** Mostrar nombre de usuario, departamento al que pertenece y su salario (asumiendo que usuarios tienen departamento_id).

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

```sql
SELECT
    u.nombre,
    d.nombre AS departamento,
    u.ingresos,
    d.presupuesto
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id
ORDER BY d.nombre, u.nombre;
```

**Explicación:**
- `u` y `d` son alias de las tablas
- `INNER JOIN` solo trae registros que existen en ambas tablas
- `ON u.departamento_id = d.id` es la condición del join
</details>

---

### 3.2 - LEFT JOIN
**Ejercicio:** Mostrar todos los usuarios y sus departamentos, aunque algunos no tengan departamento asignado.

**Dificultad:** ⭐⭐ Fácil

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `LEFT JOIN` trae todos del lado izquierdo (usuarios)
- `COALESCE` reemplaza NULL con un valor por defecto
- Esta es la diferencia clave con INNER JOIN (que excluiría los NULL)
</details>

---

### 3.3 - Múltiples JOINs
**Ejercicio:** Mostrar nombre del usuario, nombre del departamento, cantidad de proyectos asignados e horas totales.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- Múltiples LEFT JOINs se encadenan
- `COUNT(DISTINCT a.proyecto_id)` evita contar duplicados
- `NULLS LAST` pone los NULLs al final del ordenamiento
</details>

---

## NIVEL 4: Ejercicios con Window Functions

### 4.1 - ROW_NUMBER / RANK
**Ejercicio:** Numerar usuarios por ingresos dentro de cada ciudad, mostrando su ranking.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `PARTITION BY ciudad` crea una "ventana" por ciudad
- `ORDER BY ingresos DESC` ordena dentro de cada ventana
- `ROW_NUMBER`: 1,2,3,4... (única)
- `RANK`: 1,2,2,4... (salta si hay empate)
- `DENSE_RANK`: 1,2,2,3... (no salta)
</details>

---

### 4.2 - Suma acumulada
**Ejercicio:** Mostrar transacciones por usuario con suma acumulada de montos.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `ORDER BY t.fecha` sin frame especificado da suma acumulada
- `SUM(t.monto) OVER (PARTITION BY u.id)` da el total
- La segunda suma es útil para cálculos de porcentaje
</details>

---

### 4.3 - LAG / LEAD (comparar con filas anteriores)
**Ejercicio:** Mostrar cambio de ingresos mes a mes para cada usuario.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `LAG` accede a la fila anterior
- `LEAD` accede a la fila posterior
- Muy útil para análisis de tendencias
</details>

---

### 4.4 - Percentiles
**Ejercicio:** Clasificar usuarios en cuartiles según ingresos.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

```sql
SELECT
    nombre,
    ingresos,
    NTILE(4) OVER (ORDER BY ingresos) AS cuartil,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY ingresos) OVER () AS percentil_25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY ingresos) OVER () AS percentil_50,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY ingresos) OVER () AS percentil_75
FROM usuarios
WHERE ingresos IS NOT NULL
ORDER BY ingresos DESC;
```

**Explicación:**
- `NTILE(4)` divide en 4 buckets iguales
- `PERCENTILE_CONT` da el valor en un percentil específico
- Muy útil para segmentación de clientes
</details>

---

## NIVEL 5: Ejercicios con CTEs (WITH)

### 5.1 - CTE simple
**Ejercicio:** Obtener usuarios activos que tengan ingresos superiores al promedio, usando CTE.

**Dificultad:** ⭐⭐⭐ Intermedio

<details>
<summary>Ver Solución</summary>

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
- Las CTEs se definen con `WITH`
- Se ejecutan en orden (la primera, luego la segunda, etc.)
- `CROSS JOIN` con tabla de 1 fila es como tener la constante disponible
</details>

---

### 5.2 - CTEs en cascada (complejas)
**Ejercicio:** Dashboard de ventas: mostrar por ciudad el total, promedio, máximo y ranking.

**Dificultad:** ⭐⭐⭐⭐ Avanzado

<details>
<summary>Ver Solución</summary>

```sql
WITH
ventas_por_ciudad AS (
    SELECT
        u.ciudad,
        SUM(t.monto) AS total_ventas,
        COUNT(*) AS cantidad_transacciones,
        AVG(t.monto) AS promedio_venta
    FROM transacciones t
    JOIN usuarios u ON t.usuario_id = u.id
    WHERE estado = 'activo'
    GROUP BY u.ciudad
),
ranking_ciudades AS (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY total_ventas DESC) AS ranking,
        ROUND(total_ventas / SUM(total_ventas) OVER () * 100, 2) AS porcentaje_total
    FROM ventas_por_ciudad
),
clase_ciudad AS (
    SELECT
        *,
        CASE
            WHEN ranking <= 3 THEN 'Clase A (Top 3)'
            WHEN ranking <= 10 THEN 'Clase B'
            ELSE 'Clase C'
        END AS clase
    FROM ranking_ciudades
)
SELECT * FROM clase_ciudad ORDER BY ranking;
```

**Explicación:**
- Cada CTE se basa en la anterior
- La lógica más compleja se divide en pasos claros
- Mucho más legible que una sola consulta masiva
</details>

---

## NIVEL 6: Ejercicios Complejos

### 6.1 - EXISTS / NOT EXISTS
**Ejercicio:** Encontrar usuarios que tenían actividad el año pasado pero no este año.

**Dificultad:** ⭐⭐⭐⭐ Avanzado

<details>
<summary>Ver Solución</summary>

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

**Explicación:**
- `EXISTS` verifica si hay al menos una fila
- `NOT EXISTS` es la negación
- Mucho más eficiente que JOIN con GROUP BY en algunos casos
</details>

---

### 6.2 - Análisis de Cohort
**Ejercicio:** Rastrear retención de usuarios por trimestre de registro.

**Dificultad:** ⭐⭐⭐⭐⭐ Muy Avanzado

<details>
<summary>Ver Solución</summary>

```sql
WITH
registro_y_actividad AS (
    SELECT
        u.id,
        DATE_TRUNC('quarter', u.fecha_registro) AS cohort_quarter,
        DATE_TRUNC('quarter', t.fecha) AS actividad_quarter
    FROM usuarios u
    LEFT JOIN transacciones t ON u.id = t.usuario_id
),
cohort_size AS (
    SELECT
        cohort_quarter,
        COUNT(DISTINCT id) AS usuarios_cohort
    FROM registro_y_actividad
    WHERE cohort_quarter IS NOT NULL
    GROUP BY cohort_quarter
),
edad_cohort AS (
    SELECT
        rya.cohort_quarter,
        EXTRACT(QUARTER FROM (rya.actividad_quarter - rya.cohort_quarter)) AS trimestres_desde_registro,
        COUNT(DISTINCT rya.id) AS usuarios_activos,
        cs.usuarios_cohort
    FROM registro_y_actividad rya
    LEFT JOIN cohort_size cs ON rya.cohort_quarter = cs.cohort_quarter
    WHERE rya.actividad_quarter IS NOT NULL
    GROUP BY rya.cohort_quarter, EXTRACT(QUARTER FROM (rya.actividad_quarter - rya.cohort_quarter)), cs.usuarios_cohort
)
SELECT
    cohort_quarter,
    trimestres_desde_registro,
    usuarios_activos,
    usuarios_cohort,
    ROUND(usuarios_activos * 100.0 / usuarios_cohort, 1) AS retencion_porcentaje
FROM edad_cohort
ORDER BY cohort_quarter, trimestres_desde_registro;
```

**Explicación:**
- Análisis de cohortes es complejo pero muy valioso
- Muestra cómo cambia la retención a lo largo del tiempo
- Requiere múltiples pasos y ventanas de tiempo inteligentes
</details>

---

### 6.3 - Segmentación RFM
**Ejercicio:** Clasificar clientes por Recency, Frequency y Monetary value.

**Dificultad:** ⭐⭐⭐⭐⭐ Muy Avanzado

<details>
<summary>Ver Solución</summary>

```sql
WITH
rfm_base AS (
    SELECT
        u.id,
        u.nombre,
        MAX(t.fecha) AS ultima_compra,
        COUNT(DISTINCT EXTRACT(DATE FROM t.fecha)) AS dias_compra,
        COUNT(*) AS num_transacciones,
        SUM(t.monto) AS valor_total
    FROM usuarios u
    LEFT JOIN transacciones t ON u.id = t.usuario_id
    GROUP BY u.id, u.nombre
),
rfm_scores AS (
    SELECT
        *,
        EXTRACT(DAY FROM CURRENT_DATE - ultima_compra) AS dias_sin_comprar,
        NTILE(5) OVER (ORDER BY EXTRACT(DAY FROM CURRENT_DATE - ultima_compra) DESC) AS recency_score,
        NTILE(5) OVER (ORDER BY num_transacciones) AS frequency_score,
        NTILE(5) OVER (ORDER BY valor_total) AS monetary_score
    FROM rfm_base
)
SELECT
    nombre,
    dias_sin_comprar,
    num_transacciones,
    ROUND(valor_total, 2) AS valor_total,
    CASE
        WHEN recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4 THEN 'Champions'
        WHEN recency_score >= 3 AND frequency_score >= 4 THEN 'Fieles Leales'
        WHEN recency_score >= 4 AND monetary_score >= 3 THEN 'Nuevos Clientes Valiosos'
        WHEN frequency_score >= 4 AND monetary_score >= 4 THEN 'Clientes Potenciales'
        WHEN recency_score <= 2 AND frequency_score >= 3 THEN 'En Riesgo'
        WHEN recency_score <= 1 THEN 'Dormidos'
        ELSE 'Otros'
    END AS segmento
FROM rfm_scores
ORDER BY segmento, valor_total DESC;
```

**Explicación:**
- RFM es un modelo clásico de marketing
- Recency: ¿cuándo fue la última compra?
- Frequency: ¿con qué frecuencia compran?
- Monetary: ¿cuánto han gastado?
- Los resultados son accionables para marketing
</details>

---

## Desafíos Extras

### Desafío 1: Crecimiento Mes a Mes
**Ejercicio:** Calcular el crecimiento porcentual de transacciones por mes.

<details>
<summary>Ver Solución</summary>

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
    LAG(num_transacciones) OVER (ORDER BY mes) AS transacciones_mes_anterior,
    ROUND(
        ((num_transacciones - LAG(num_transacciones) OVER (ORDER BY mes))
        / LAG(num_transacciones) OVER (ORDER BY mes) * 100),
        2
    ) AS crecimiento_porcentaje
FROM transacciones_mensuales
ORDER BY mes DESC;
```
</details>

---

### Desafío 2: Top Productos por Categoría
**Ejercicio:** Mostrar los top 3 productos más vendidos dentro de cada categoría.

<details>
<summary>Ver Solución</summary>

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
</details>

---

**¡Practica estos ejercicios y dominarás SQL!**
