# Roadmap de Aprendizaje SQL

## ¿Cómo usar este roadmap?

Este roadmap está diseñado para que avances de forma progresiva. Cada semana tienes metas claras:

- **Semana 1-2**: Fundamentos sólidos
- **Semana 3-4**: Queries prácticas
- **Semana 5-6**: Análisis complejos
- **Semana 7-8**: Optimización y especialización

---

## SEMANA 1: Fundamentos (5-7 días)

### Día 1: SELECT, WHERE, ORDER BY, LIMIT
**Objetivo**: Poder extraer datos de forma básica

```sql
-- Ejercicio 1: TOP 10 usuarios más nuevos
SELECT id, nombre, email, fecha_registro
FROM usuarios
ORDER BY fecha_registro DESC
LIMIT 10;

-- Ejercicio 2: Usuarios de una ciudad específica
SELECT * FROM usuarios
WHERE ciudad = 'Madrid'
ORDER BY nombre;

-- Ejercicio 3: Usuarios entre 25 y 35 años de edad
SELECT nombre, edad, ciudad
FROM usuarios
WHERE edad BETWEEN 25 AND 35
ORDER BY edad;
```

**Tareas:**
- [ ] Ejecutar las 3 queries en tu base de datos
- [ ] Modificarlas para diferentes ciudades y edades
- [ ] Practicar LIKE para búsquedas: `nombre LIKE 'J%'`

---

### Día 2: Agregaciones (COUNT, SUM, AVG, MIN, MAX)
**Objetivo**: Entender cómo llevar operaciones de matemáticas

```sql
-- Ejercicio 1: Estadísticas de edad
SELECT
    COUNT(*) AS total_usuarios,
    AVG(edad) AS edad_promedio,
    MIN(edad) AS edad_minima,
    MAX(edad) AS edad_maxima
FROM usuarios;

-- Ejercicio 2: Cuántas transacciones hay
SELECT COUNT(*) AS total_transacciones FROM transacciones;

-- Ejercicio 3: Ingresos totales y promedio
SELECT
    SUM(ingresos) AS total_ingresos,
    AVG(ingresos) AS ingreso_promedio
FROM usuarios
WHERE estado = 'activo';
```

**Tareas:**
- [ ] Ejecutar los 3 ejercicios
- [ ] Agregar máximos y mínimos a cada query
- [ ] Usar ROUND para redondear decimales

---

### Día 3: GROUP BY y HAVING
**Objetivo**: Agrupar datos y contar elementos

```sql
-- Ejercicio 1: Cuántos usuarios por ciudad
SELECT
    ciudad,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
ORDER BY cantidad_usuarios DESC;

-- Ejercicio 2: Edad promedio por ciudad
SELECT
    ciudad,
    AVG(edad) AS edad_promedio,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
HAVING COUNT(*) > 5
ORDER BY edad_promedio DESC;

-- Ejercicio 3: Estados con ingreso promedio > 50000
SELECT
    estado,
    AVG(ingresos) AS ingresos_promedio,
    COUNT(*) AS cantidad
FROM usuarios
WHERE ingresos IS NOT NULL
GROUP BY estado
HAVING AVG(ingresos) > 50000;
```

**Tareas:**
- [ ] Ejecutar los 3 ejercicios
- [ ] Cambiar el filtro HAVING a diferentes valores
- [ ] Agregar más columnas al GROUP BY

---

### Día 4: DISTINCT y CASE
**Objetivo**: Valores únicos y lógica condicional

```sql
-- Ejercicio 1: Ciudades únicas
SELECT DISTINCT ciudad FROM usuarios ORDER BY ciudad;

-- Ejercicio 2: Combinaciones únicas de ciudad-estado
SELECT DISTINCT ciudad, estado FROM usuarios ORDER BY ciudad, estado;

-- Ejercicio 3: Categorizar usuarios por edad
SELECT
    nombre,
    edad,
    CASE
        WHEN edad < 18 THEN 'Menor'
        WHEN edad < 30 THEN 'Joven'
        WHEN edad < 60 THEN 'Adulto'
        ELSE 'Mayor'
    END AS categoria
FROM usuarios
ORDER BY edad;

-- Ejercicio 4: Contar por categoría
SELECT
    CASE
        WHEN edad < 18 THEN 'Menor'
        WHEN edad < 30 THEN 'Joven'
        WHEN edad < 60 THEN 'Adulto'
        ELSE 'Mayor'
    END AS categoria,
    COUNT(*) AS cantidad
FROM usuarios
GROUP BY categoria;
```

**Tareas:**
- [ ] Ejecutar los 4 ejercicios
- [ ] Crear categorías de ingresos (Bajo/Medio/Alto)
- [ ] Combinar CASE con GROUP BY

---

### Día 5: Funciones de Texto y Fecha
**Objetivo**: Manipular texto y fechas

```sql
-- Ejercicio 1: Mayúsculas y minúsculas
SELECT
    nombre,
    UPPER(nombre) AS mayusculas,
    LOWER(nombre) AS minusculas,
    LENGTH(nombre) AS largo
FROM usuarios
LIMIT 10;

-- Ejercicio 2: Función SUBSTR
SELECT
    email,
    SUBSTR(email, 1, POSITION('@' IN email) - 1) AS usuario,
    SUBSTR(email, POSITION('@' IN email) + 1) AS dominio
FROM usuarios
LIMIT 10;

-- Ejercicio 3: Operaciones de fecha
SELECT
    nombre,
    fecha_registro,
    CURRENT_DATE AS hoy,
    CURRENT_DATE - fecha_registro AS dias_registrado
FROM usuarios
ORDER BY fecha_registro DESC
LIMIT 10;

-- Ejercicio 4: Extraer partes de fecha
SELECT
    fecha_registro,
    EXTRACT(YEAR FROM fecha_registro) AS año,
    EXTRACT(MONTH FROM fecha_registro) AS mes,
    EXTRACT(DAY FROM fecha_registro) AS dia
FROM usuarios
LIMIT 10;
```

**Tareas:**
- [ ] Ejecutar los 4 ejercicios
- [ ] Extraer dominio de email para todos los usuarios
- [ ] Calcular cuántos días/años llevan registrados los usuarios

---

## SEMANA 2: JOINs Básicos (5-7 días)

### Día 6: INNER JOIN
**Objetivo**: Conectar dos tablas

```sql
-- Teniendo tablas:
-- usuarios (id, nombre, departamento_id)
-- departamentos (id, nombre)

-- Ejercicio 1: Usuarios con su departamento
SELECT
    u.nombre AS usuario,
    d.nombre AS departamento
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id;

-- Ejercicio 2: Contar usuarios por departamento
SELECT
    d.nombre AS departamento,
    COUNT(u.id) AS cantidad_usuarios
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id
GROUP BY d.id, d.nombre
ORDER BY cantidad_usuarios DESC;

-- Ejercicio 3: Usuarios y transacciones
SELECT
    u.nombre AS usuario,
    COUNT(t.id) AS total_transacciones,
    SUM(t.monto) AS total_gastado
FROM usuarios u
INNER JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre;
```

**Tareas:**
- [ ] Ejecutar los 3 ejercicios
- [ ] INNER JOIN solo trae registros que existen en AMBAS tablas
- [ ] Practicar con diferentes tablas

---

### Día 7: LEFT JOIN
**Objetivo**: Mantener todos del lado izquierdo, inclusos sin match

```sql
-- Ejercicio 1: Usuarios aunque NO tengan departamento
SELECT
    u.nombre,
    COALESCE(d.nombre, 'Sin asignar') AS departamento
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
ORDER BY u.nombre;

-- Ejercicio 2: Usuarios sin transacciones
SELECT
    u.nombre,
    COUNT(t.id) AS total_transacciones
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre
HAVING COUNT(t.id) = 0
ORDER BY u.nombre;

-- Ejercicio 3: Dashboard de usuarios
SELECT
    u.nombre,
    u.email,
    COALESCE(COUNT(DISTINCT t.id), 0) AS transacciones,
    COALESCE(SUM(t.monto), 0) AS monto_total
FROM usuarios u
LEFT JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre, u.email
ORDER BY monto_total DESC;
```

**Tareas:**
- [ ] Entender diferencia entre INNER y LEFT
- [ ] Usar COALESCE para manejar NULLs
- [ ] Practicar HAVING con LEFT JOINs

---

### Día 8: Múltiples JOINs
**Objetivo**: Conectar 3+ tablas

```sql
-- Ejercicio 1: Usuarios, departamentos y proyectos
SELECT
    u.nombre AS usuario,
    d.nombre AS departamento,
    p.nombre AS proyecto
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
LEFT JOIN proyectos p ON d.id = p.departamento_id
ORDER BY u.nombre, p.nombre;

-- Ejercicio 2: Análisis completo
SELECT
    u.nombre AS usuario,
    d.nombre AS departamento,
    COUNT(DISTINCT p.id) AS proyectos,
    COUNT(DISTINCT t.id) AS transacciones,
    SUM(t.monto) AS monto_total
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
LEFT JOIN proyectos p ON d.id = p.departamento_id
LEFT JOIN transacciones t ON u.id = t.usuario_id
GROUP BY u.id, u.nombre, d.id, d.nombre
ORDER BY monto_total DESC NULLS LAST;
```

**Tareas:**
- [ ] Ejecutar los ejercicios
- [ ] Entender el orden de los JOINs
- [ ] Agregar un cuarto JOIN

---

## SEMANA 3-4: Queries Intermedias y Prácticas

### Día 9-10: Window Functions Básicas
**Objetivo**: Ranking y sumas acumuladas

```sql
-- Ejercicio 1: Ranking de usuarios por ingresos en su ciudad
SELECT
    nombre,
    ciudad,
    ingresos,
    ROW_NUMBER() OVER (PARTITION BY ciudad ORDER BY ingresos DESC) AS ranking
FROM usuarios
WHERE estado = 'activo'
ORDER BY ciudad, ranking;

-- Ejercicio 2: Comparar con promedio del departamento
SELECT
    nombre,
    departamento_id,
    ingresos,
    AVG(ingresos) OVER (PARTITION BY departamento_id) AS prom_depto,
    ingresos - AVG(ingresos) OVER (PARTITION BY departamento_id) AS diferencia
FROM usuarios
ORDER BY departamento_id, ingresos DESC;

-- Ejercicio 3: Suma acumulada de transacciones
SELECT
    u.nombre,
    t.fecha,
    t.monto,
    SUM(t.monto) OVER (PARTITION BY u.id ORDER BY t.fecha) AS suma_acumulada
FROM transacciones t
JOIN usuarios u ON t.usuario_id = u.id
ORDER BY u.id, t.fecha;
```

**Tareas:**
- [ ] Entender PARTITION BY y ORDER BY en window functions
- [ ] Practicar ROW_NUMBER, RANK, DENSE_RANK
- [ ] Crear suma acumulada para diferentes usuarios

---

### Día 11-12: CTEs (WITH)
**Objetivo**: Estructurar queries complejas

```sql
-- Ejercicio 1: CTE simple
WITH usuarios_activos AS (
    SELECT id, nombre, ingresos
    FROM usuarios
    WHERE estado = 'activo'
)
SELECT * FROM usuarios_activos
WHERE ingresos > 50000
ORDER BY ingresos DESC;

-- Ejercicio 2: CTEs en cascada
WITH
usuarios_activos AS (
    SELECT id, nombre, departamento_id, ingresos
    FROM usuarios
    WHERE estado = 'activo'
),
prom_por_depto AS (
    SELECT
        departamento_id,
        AVG(ingresos) AS promedio
    FROM usuarios_activos
    GROUP BY departamento_id
)
SELECT
    ua.nombre,
    ua.ingresos,
    ppd.promedio,
    ua.ingresos - ppd.promedio AS diferencia
FROM usuarios_activos ua
JOIN prom_por_depto ppd ON ua.departamento_id = ppd.departamento_id
WHERE ua.ingresos > ppd.promedio
ORDER BY diferencia DESC;

-- Ejercicio 3: Dashboard con CTEs
WITH ventas_mensuales AS (
    SELECT
        u.id,
        u.nombre,
        DATE_TRUNC('month', t.fecha) AS mes,
        SUM(t.monto) AS total_mes
    FROM usuarios u
    LEFT JOIN transacciones t ON u.id = t.usuario_id
    GROUP BY u.id, u.nombre, DATE_TRUNC('month', t.fecha)
)
SELECT
    nombre,
    mes,
    total_mes,
    LAG(total_mes) OVER (PARTITION BY id ORDER BY mes) AS mes_anterior,
    total_mes - LAG(total_mes) OVER (PARTITION BY id ORDER BY mes) AS cambio
FROM ventas_mensuales
WHERE mes IS NOT NULL
ORDER BY nombre, mes;
```

**Tareas:**
- [ ] Crear 5 CTEs diferentes
- [ ] Combinar CTEs con JOINs y window functions
- [ ] Documentar queries con CTEs

---

## SEMANA 5-6: Análisis Complejos

### Proyecto 1: Dashboard de RH
```sql
-- Análisis completo de empleados
WITH empleados_info AS (
    SELECT
        e.id,
        e.nombre,
        d.nombre AS departamento,
        e.salario,
        DATE_PART('year', AGE(e.fecha_nacimiento)) AS edad,
        e.estado
    FROM empleados e
    LEFT JOIN departamentos d ON e.departamento_id = d.id
),
rankings AS (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY departamento ORDER BY salario DESC) AS ranking_salario,
        AVG(salario) OVER (PARTITION BY departamento) AS salario_promedio_depto
    FROM empleados_info
    WHERE estado = 'activo'
)
SELECT
    nombre,
    departamento,
    salario,
    ranking_salario,
    salario_promedio_depto,
    salario - salario_promedio_depto AS diferencia,
    CASE
        WHEN ranking_salario <= 3 THEN 'Top 3'
        WHEN salario > salario_promedio_depto THEN 'Sobre promedio'
        ELSE 'Bajo promedio'
    END AS categoria
FROM rankings
ORDER BY departamento, ranking_salario;
```

### Proyecto 2: Análisis de Ventas
```sql
-- Tendencias de ventas
WITH ventas_diarias AS (
    SELECT
        DATE(fecha) AS dia,
        SUM(monto) AS total_dia,
        COUNT(*) AS transacciones
    FROM transacciones
    GROUP BY DATE(fecha)
),
tendencias AS (
    SELECT
        dia,
        total_dia,
        LAG(total_dia) OVER (ORDER BY dia) AS dia_anterior,
        AVG(total_dia) OVER (ORDER BY dia ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS promedio_7dias,
        ROUND(((total_dia - LAG(total_dia) OVER (ORDER BY dia))
            / LAG(total_dia) OVER (ORDER BY dia) * 100), 2) AS cambio_porcentaje
    FROM ventas_diarias
)
SELECT
    dia,
    total_dia,
    promedio_7dias,
    cambio_porcentaje,
    CASE
        WHEN cambio_porcentaje > 10 THEN 'Crecimiento fuerte'
        WHEN cambio_porcentaje > 0 THEN 'Crecimiento'
        WHEN cambio_porcentaje > -10 THEN 'Caída leve'
        ELSE 'Caída fuerte'
    END AS tendencia
FROM tendencias
WHERE dia >= CURRENT_DATE - INTERVAL '3 months'
ORDER BY dia DESC;
```

### Proyecto 3: Segmentación RFM
```sql
-- Clasificar clientes
WITH rfm_base AS (
    SELECT
        u.id,
        u.nombre,
        MAX(t.fecha) AS ultima_compra,
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
        NTILE(5) OVER (ORDER BY EXTRACT(DAY FROM CURRENT_DATE - ultima_compra) DESC) AS recency_tier,
        NTILE(5) OVER (ORDER BY num_transacciones) AS frequency_tier,
        NTILE(5) OVER (ORDER BY valor_total) AS monetary_tier
    FROM rfm_base
)
SELECT
    nombre,
    dias_sin_comprar,
    num_transacciones,
    ROUND(valor_total, 2) AS valor_total,
    CASE
        WHEN recency_tier >= 4 AND frequency_tier >= 4 AND monetary_tier >= 4 THEN 'Champions'
        WHEN recency_tier >= 3 AND frequency_tier >= 4 THEN 'Fieles'
        WHEN recency_tier >= 4 AND monetary_tier >= 3 THEN 'Nuevos Valiosos'
        WHEN frequency_tier >= 4 AND monetary_tier >= 4 THEN 'Potencial'
        WHEN recency_tier <= 2 AND frequency_tier >= 3 THEN 'En Riesgo'
        WHEN recency_tier <= 1 THEN 'Dormidos'
        ELSE 'Otros'
    END AS segmento
FROM rfm_scores
WHERE valor_total IS NOT NULL
ORDER BY segmento, valor_total DESC;
```

---

## SEMANA 7-8: Optimización y Especialización

### Optimización de Queries
1. **Usar EXPLAIN** para entender planes de ejecución
2. **Crear índices** en columnas WHERE y JOINs frecuentes
3. **Evitar SELECT *** (en BigQuery especialmente)
4. **Usar CTEs** para organizar lógica compleja
5. **Particionar tablas** en BigQuery

### Próximos Pasos
- [ ] Aprender PostgreSQL específico (Arrays, JSONB, etc.)
- [ ] Aprender BigQuery específico (particiones, clustering, etc.)
- [ ] Implementar en producción
- [ ] Crear dashboards con los datos
- [ ] Aprender ETL (Extract, Transform, Load)

---

## Recursos por Semana

| Semana | Tema | Recurso |
|--------|------|---------|
| 1 | Fundamentos | PgExercises.com capítulos 1-5 |
| 2 | JOINs | PgExercises.com capítulos 6-8 |
| 3-4 | Window Functions | Mode SQL Tutorial |
| 5-6 | Análisis Complejos | LeetCode Database |
| 7-8 | Optimización | PostgreSQL/BigQuery docs |

---

**¡Dedica 1-2 horas diarias y en 8 semanas serás experto en SQL!**
