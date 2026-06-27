# Guía Completa de SQL - De Básico a Avanzado

## Tabla de Contenidos
1. [Nivel 1: Fundamentos Básicos](#nivel-1-fundamentos-básicos)
2. [Nivel 2: Consultas Intermedias](#nivel-2-consultas-intermedias)
3. [Nivel 3: Joins y Relaciones](#nivel-3-joins-y-relaciones)
4. [Nivel 4: Funciones Avanzadas](#nivel-4-funciones-avanzadas)
5. [Nivel 5: Window Functions (Ventanas)](#nivel-5-window-functions-ventanas)
6. [Nivel 6: CTEs y Subconsultas Complejas](#nivel-6-ctes-y-subconsultas-complejas)
7. [Nivel 7: Características Avanzadas de PostgreSQL y BigQuery](#nivel-7-características-avanzadas)

---

## NIVEL 1: Fundamentos Básicos

### 1.1 - SELECT básico
La consulta más simple: traer datos de una tabla
```sql
-- Traer todas las columnas
SELECT * FROM usuarios;

-- Traer columnas específicas
SELECT id, nombre, email FROM usuarios;

-- Dar alias a las columnas en el resultado
SELECT
    id AS usuario_id,
    nombre AS nombre_completo,
    email AS correo_electronico
FROM usuarios;
```

### 1.2 - WHERE: Filtrar datos
```sql
-- Igualdad
SELECT * FROM usuarios WHERE edad = 30;

-- Mayor que / Menor que
SELECT * FROM usuarios WHERE edad > 25;

-- BETWEEN (entre dos valores)
SELECT * FROM usuarios WHERE edad BETWEEN 25 AND 35;

-- IN (valor en una lista)
SELECT * FROM usuarios WHERE estado IN ('activo', 'pendiente');

-- NOT IN
SELECT * FROM usuarios WHERE estado NOT IN ('eliminado', 'suspendido');

-- LIKE (búsquedas de texto)
SELECT * FROM usuarios WHERE nombre LIKE 'Juan%';  -- Empieza con Juan
SELECT * FROM usuarios WHERE email LIKE '%@gmail.com';  -- Termina con @gmail.com

-- IS NULL / IS NOT NULL
SELECT * FROM usuarios WHERE fecha_eliminacion IS NULL;  -- Registros no eliminados
SELECT * FROM usuarios WHERE numero_telefono IS NOT NULL;
```

### 1.3 - ORDER BY: Ordenar resultados
```sql
-- Orden ascendente (por defecto)
SELECT * FROM usuarios ORDER BY nombre;

-- Orden descendente
SELECT * FROM usuarios ORDER BY edad DESC;

-- Múltiples columnas
SELECT * FROM usuarios ORDER BY edad DESC, nombre ASC;
```

### 1.4 - LIMIT: Limitar resultados
```sql
-- Top 10 usuarios más nuevos
SELECT * FROM usuarios ORDER BY fecha_registro DESC LIMIT 10;

-- Los 5 usuarios más viejos
SELECT * FROM usuarios ORDER BY edad ASC LIMIT 5;

-- Pagination: saltar 20 registros y traer 10 (página 3)
SELECT * FROM usuarios LIMIT 10 OFFSET 20;
```

### 1.5 - DISTINCT: Valores únicos
```sql
-- Estados únicos en la tabla
SELECT DISTINCT estado FROM usuarios;

-- Combinación única de ciudad y estado
SELECT DISTINCT ciudad, estado FROM usuarios;
```

---

## NIVEL 2: Consultas Intermedias

### 2.1 - Operadores lógicos (AND, OR, NOT)
```sql
-- AND: deben cumplirse ambas condiciones
SELECT * FROM usuarios
WHERE edad > 25 AND estado = 'activo';

-- OR: al menos una condición
SELECT * FROM usuarios
WHERE edad < 18 OR edad > 65;

-- Combinación compleja
SELECT * FROM usuarios
WHERE (edad >= 25 AND edad <= 35)
  AND (estado = 'activo' OR estado = 'pendiente')
  AND ciudad NOT IN ('Cancelada', 'Eliminada');
```

### 2.2 - Funciones de Agregación (COUNT, SUM, AVG, MIN, MAX)
```sql
-- Contar registros
SELECT COUNT(*) AS total_usuarios FROM usuarios;

-- Contar valores no nulos
SELECT COUNT(numero_telefono) AS usuarios_con_telefono FROM usuarios;

-- Sumar valores
SELECT SUM(saldo) AS saldo_total FROM cuentas;

-- Promedio
SELECT AVG(edad) AS edad_promedio FROM usuarios;

-- Mínimo y máximo
SELECT
    MIN(edad) AS edad_minima,
    MAX(edad) AS edad_maxima
FROM usuarios;
```

### 2.3 - GROUP BY: Agrupar datos
```sql
-- Contar usuarios por estado
SELECT estado, COUNT(*) AS cantidad FROM usuarios GROUP BY estado;

-- Promedio de edad por ciudad
SELECT
    ciudad,
    AVG(edad) AS edad_promedio,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
ORDER BY edad_promedio DESC;

-- Múltiples agrupaciones
SELECT
    estado,
    ciudad,
    COUNT(*) AS cantidad,
    AVG(edad) AS edad_promedio
FROM usuarios
GROUP BY estado, ciudad
ORDER BY estado, ciudad;
```

### 2.4 - HAVING: Filtrar grupos (después de GROUP BY)
```sql
-- Ciudades con más de 100 usuarios
SELECT
    ciudad,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
GROUP BY ciudad
HAVING COUNT(*) > 100;

-- Estados con edad promedio mayor a 30
SELECT
    estado,
    AVG(edad) AS edad_promedio
FROM usuarios
GROUP BY estado
HAVING AVG(edad) > 30;

-- Ciudades donde el promedio de ingresos supera 50000
SELECT
    ciudad,
    AVG(ingresos) AS ingresos_promedio,
    COUNT(*) AS cantidad_usuarios
FROM usuarios
WHERE ingresos IS NOT NULL
GROUP BY ciudad
HAVING AVG(ingresos) > 50000
ORDER BY ingresos_promedio DESC;
```

### 2.5 - CASE: Lógica condicional
```sql
-- Categorizar por edad
SELECT
    nombre,
    edad,
    CASE
        WHEN edad < 18 THEN 'Menor de edad'
        WHEN edad BETWEEN 18 AND 65 THEN 'Adulto activo'
        ELSE 'Jubilado'
    END AS categoria_edad
FROM usuarios;

-- Múltiples condiciones
SELECT
    nombre,
    ingresos,
    CASE
        WHEN ingresos < 20000 THEN 'Bajo'
        WHEN ingresos < 50000 THEN 'Medio'
        WHEN ingresos < 100000 THEN 'Alto'
        ELSE 'Muy alto'
    END AS nivel_ingresos
FROM usuarios;

-- CASE con agregación
SELECT
    ciudad,
    COUNT(CASE WHEN edad < 30 THEN 1 END) AS menores_30,
    COUNT(CASE WHEN edad >= 30 AND edad < 50 THEN 1 END) AS entre_30_50,
    COUNT(CASE WHEN edad >= 50 THEN 1 END) AS mayores_50
FROM usuarios
GROUP BY ciudad;
```

---

## NIVEL 3: Joins y Relaciones

Tabla de ejemplo para joins:
```
Tabla: usuarios (id, nombre, departamento_id)
Tabla: departamentos (id, nombre)
Tabla: proyectos (id, nombre, departamento_id)
Tabla: asignaciones (id, usuario_id, proyecto_id, horas)
```

### 3.1 - INNER JOIN (intersección)
```sql
-- Traer usuarios con su departamento
SELECT
    u.id,
    u.nombre AS usuario,
    d.nombre AS departamento
FROM usuarios u
INNER JOIN departamentos d ON u.departamento_id = d.id;
```

### 3.2 - LEFT JOIN (todos del lado izquierdo)
```sql
-- Usuarios y sus departamentos (incluso sin departamento asignado)
SELECT
    u.id,
    u.nombre,
    d.nombre AS departamento
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id;
```

### 3.3 - RIGHT JOIN (todos del lado derecho)
```sql
-- Todos los departamentos y usuarios que tienen
SELECT
    d.nombre AS departamento,
    COUNT(u.id) AS cantidad_usuarios
FROM usuarios u
RIGHT JOIN departamentos d ON u.departamento_id = d.id
GROUP BY d.id, d.nombre;
```

### 3.4 - FULL OUTER JOIN (todos ambos lados)
```sql
-- Todos los usuarios y departamentos (PostgreSQL)
SELECT
    COALESCE(u.nombre, 'Sin usuario') AS usuario,
    COALESCE(d.nombre, 'Sin departamento') AS departamento
FROM usuarios u
FULL OUTER JOIN departamentos d ON u.departamento_id = d.id;
```

### 3.5 - CROSS JOIN (producto cartesiano)
```sql
-- Todas las combinaciones posibles
SELECT
    u.nombre,
    p.nombre
FROM usuarios u
CROSS JOIN proyectos p;
```

### 3.6 - Múltiples JOINs (complejo)
```sql
-- Usuarios, departamentos y cantidad de proyectos asignados
SELECT
    u.id,
    u.nombre AS usuario,
    d.nombre AS departamento,
    COUNT(DISTINCT a.proyecto_id) AS proyectos_asignados,
    SUM(a.horas) AS total_horas
FROM usuarios u
LEFT JOIN departamentos d ON u.departamento_id = d.id
LEFT JOIN asignaciones a ON u.id = a.usuario_id
GROUP BY u.id, u.nombre, d.id, d.nombre
ORDER BY total_horas DESC NULLS LAST;
```

### 3.7 - Self Join (tabla con ella misma)
```sql
-- Tabla de empleados con gerente
-- Tabla: empleados (id, nombre, gerente_id)

SELECT
    e.nombre AS empleado,
    g.nombre AS gerente,
    e.departamento_id
FROM empleados e
LEFT JOIN empleados g ON e.gerente_id = g.id
ORDER BY e.gerente_id;
```

---

## NIVEL 4: Funciones Avanzadas

### 4.1 - Funciones de Texto (PostgreSQL)
```sql
-- UPPER / LOWER / INITCAP
SELECT
    UPPER(nombre) AS mayusculas,
    LOWER(nombre) AS minusculas,
    INITCAP(nombre) AS capitalizadas
FROM usuarios;

-- LENGTH / SUBSTR
SELECT
    nombre,
    LENGTH(nombre) AS largo,
    SUBSTR(nombre, 1, 5) AS primeros_5_caracteres
FROM usuarios;

-- CONCAT / ||
SELECT
    nombre || ' ' || apellido AS nombre_completo,
    CONCAT(nombre, ' (', email, ')') AS nombre_con_email
FROM usuarios;

-- REPLACE / TRIM
SELECT
    nombre,
    REPLACE(nombre, 'Juan', 'Jonathan') AS nombre_reemplazado,
    TRIM('  ' || nombre || '  ') AS sin_espacios
FROM usuarios;

-- SPLIT_PART (PostgreSQL específico)
SELECT
    email,
    SPLIT_PART(email, '@', 1) AS usuario,
    SPLIT_PART(email, '@', 2) AS dominio
FROM usuarios;
```

### 4.2 - Funciones de Fecha (PostgreSQL)
```sql
-- CURRENT_DATE / CURRENT_TIMESTAMP / NOW()
SELECT
    nombre,
    fecha_registro,
    CURRENT_DATE AS hoy,
    CURRENT_TIMESTAMP AS ahora
FROM usuarios;

-- Cálculos de fechas
SELECT
    nombre,
    fecha_nacimiento,
    AGE(CURRENT_DATE, fecha_nacimiento) AS edad_calculada,
    DATE_PART('year', AGE(CURRENT_DATE, fecha_nacimiento)) AS años
FROM usuarios;

-- EXTRACT
SELECT
    nombre,
    fecha_registro,
    EXTRACT(YEAR FROM fecha_registro) AS año,
    EXTRACT(MONTH FROM fecha_registro) AS mes,
    EXTRACT(DAY FROM fecha_registro) AS día
FROM usuarios;

-- DATE_TRUNC (redondear fechas)
SELECT
    DATE_TRUNC('month', fecha_registro) AS mes,
    COUNT(*) AS registros_mes
FROM usuarios
GROUP BY DATE_TRUNC('month', fecha_registro)
ORDER BY mes DESC;
```

### 4.3 - Funciones Matemáticas
```sql
-- ROUND / CEIL / FLOOR / ABS
SELECT
    nombre,
    ingresos,
    ROUND(ingresos, 2) AS redondeado,
    CEIL(ingresos) AS arriba,
    FLOOR(ingresos) AS abajo,
    ABS(ingresos - 50000) AS diferencia_50k
FROM usuarios;

-- POWER / SQRT
SELECT
    nombre,
    edad,
    POWER(edad, 2) AS edad_al_cuadrado,
    SQRT(ingresos) AS raiz_ingresos
FROM usuarios;
```

### 4.4 - Funciones de String Matching (REGEX)
```sql
-- PostgreSQL
SELECT
    email,
    CASE
        WHEN email ~ '.*@gmail\.com$' THEN 'Gmail'
        WHEN email ~ '.*@hotmail\..*$' THEN 'Hotmail'
        WHEN email ~ '.*@yahoo\..*$' THEN 'Yahoo'
        ELSE 'Otro'
    END AS proveedor_email
FROM usuarios;

-- Extraer números de un string
SELECT
    nombre,
    (regexp_matches(nombre, '\d+', 'g'))[1] AS numero
FROM usuarios
WHERE nombre ~ '\d+';
```

---

## NIVEL 5: Window Functions (Ventanas)

Las window functions son EXTREMADAMENTE poderosas y comunes en PostgreSQL y BigQuery.

### 5.1 - ROW_NUMBER / RANK / DENSE_RANK
```sql
-- Numerar filas ordenadas por edad
SELECT
    nombre,
    edad,
    ROW_NUMBER() OVER (ORDER BY edad DESC) AS numero_fila,
    RANK() OVER (ORDER BY edad DESC) AS rango,
    DENSE_RANK() OVER (ORDER BY edad DESC) AS rango_denso
FROM usuarios;

-- Top 3 usuarios más pagados por departamento
SELECT *
FROM (
    SELECT
        nombre,
        departamento_id,
        ingresos,
        ROW_NUMBER() OVER (PARTITION BY departamento_id ORDER BY ingresos DESC) AS rango
    FROM usuarios
) ranked
WHERE rango <= 3;
```

### 5.2 - LAG / LEAD (acceder a filas anterior/posterior)
```sql
-- Comparar ingresos con el mes anterior
SELECT
    mes,
    ingresos,
    LAG(ingresos) OVER (ORDER BY mes) AS ingresos_mes_anterior,
    ingresos - LAG(ingresos) OVER (ORDER BY mes) AS cambio,
    ROUND(((ingresos - LAG(ingresos) OVER (ORDER BY mes))
        / LAG(ingresos) OVER (ORDER BY mes) * 100), 2) AS cambio_porcentaje
FROM ingresos_mensuales
ORDER BY mes;
```

### 5.3 - SUM / AVG OVER (agregación con ventana)
```sql
-- Suma acumulada (running total)
SELECT
    nombre,
    fecha,
    venta,
    SUM(venta) OVER (ORDER BY fecha) AS suma_acumulada,
    AVG(venta) OVER (ORDER BY fecha) AS promedio_acumulado
FROM ventas
ORDER BY fecha;

-- Promedio de las últimas 3 ventas
SELECT
    nombre,
    fecha,
    venta,
    AVG(venta) OVER (
        ORDER BY fecha
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS promedio_ultimas_3
FROM ventas
ORDER BY nombre, fecha;
```

### 5.4 - PARTITION BY (ventanas separadas)
```sql
-- Promedio de edad por departamento
SELECT
    nombre,
    departamento_id,
    edad,
    AVG(edad) OVER (PARTITION BY departamento_id) AS edad_promedio_departamento,
    edad - AVG(edad) OVER (PARTITION BY departamento_id) AS diferencia_promedio
FROM usuarios;

-- Porcentaje respecto al total por categoría
SELECT
    categoria,
    nombre,
    ventas,
    SUM(ventas) OVER (PARTITION BY categoria) AS total_categoria,
    ROUND((ventas / SUM(ventas) OVER (PARTITION BY categoria) * 100), 2) AS porcentaje
FROM ventas_por_producto;
```

### 5.5 - FIRST_VALUE / LAST_VALUE
```sql
-- Comparar con el mejor y peor mes del año
SELECT
    mes,
    ingresos,
    FIRST_VALUE(ingresos) OVER (ORDER BY mes) AS primer_mes,
    LAST_VALUE(ingresos) OVER (
        ORDER BY mes
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS ultimo_mes,
    ingresos - FIRST_VALUE(ingresos) OVER (ORDER BY mes) AS diferencia_inicio
FROM ingresos_mensuales;
```

### 5.6 - NTILE (distribución en buckets)
```sql
-- Dividir usuarios en 4 cuartiles por ingresos
SELECT
    nombre,
    ingresos,
    NTILE(4) OVER (ORDER BY ingresos) AS cuartil
FROM usuarios
ORDER BY cuartil, ingresos;
```

---

## NIVEL 6: CTEs y Subconsultas Complejas

### 6.1 - WITH (Common Table Expressions)
```sql
-- Estructura básica
WITH usuarios_activos AS (
    SELECT id, nombre, email
    FROM usuarios
    WHERE estado = 'activo'
)
SELECT * FROM usuarios_activos WHERE nombre LIKE 'J%';
```

### 6.2 - CTEs en cascada (múltiples)
```sql
WITH
-- Paso 1: Filtrar usuarios activos
usuarios_activos AS (
    SELECT id, nombre, departamento_id, ingresos
    FROM usuarios
    WHERE estado = 'activo'
),
-- Paso 2: Calcular ingresos por departamento
ingresos_departamento AS (
    SELECT
        departamento_id,
        AVG(ingresos) AS ingreso_promedio,
        MAX(ingresos) AS ingreso_maximo
    FROM usuarios_activos
    GROUP BY departamento_id
),
-- Paso 3: Combinar
resultado_final AS (
    SELECT
        ua.nombre,
        ua.ingresos,
        id.ingreso_promedio,
        id.ingreso_maximo,
        ua.ingresos - id.ingreso_promedio AS diferencia
    FROM usuarios_activos ua
    JOIN ingresos_departamento id ON ua.departamento_id = id.departamento_id
)
SELECT * FROM resultado_final ORDER BY diferencia DESC;
```

### 6.3 - Subconsultas en SELECT
```sql
-- Obtener el promedio general y comparar
SELECT
    nombre,
    departamento_id,
    ingresos,
    (SELECT AVG(ingresos) FROM usuarios) AS promedio_general,
    ingresos - (SELECT AVG(ingresos) FROM usuarios) AS diferencia_promedio
FROM usuarios
WHERE ingresos > (SELECT AVG(ingresos) FROM usuarios);
```

### 6.4 - Subconsultas en FROM
```sql
-- Usuarios con ingresos superiores al promedio de su departamento
SELECT
    resultado.nombre,
    resultado.ingresos,
    resultado.promedio_departamento
FROM (
    SELECT
        u.nombre,
        u.ingresos,
        AVG(u.ingresos) OVER (PARTITION BY u.departamento_id) AS promedio_departamento
    FROM usuarios u
) resultado
WHERE resultado.ingresos > resultado.promedio_departamento;
```

### 6.5 - Subconsultas correlacionadas
```sql
-- Traer el usuario más nuevo de cada departamento
SELECT
    u.id,
    u.nombre,
    u.departamento_id,
    u.fecha_registro
FROM usuarios u
WHERE u.fecha_registro = (
    SELECT MAX(u2.fecha_registro)
    FROM usuarios u2
    WHERE u2.departamento_id = u.departamento_id
);
```

### 6.6 - EXISTS / NOT EXISTS
```sql
-- Departamentos que tienen usuarios
SELECT
    d.id,
    d.nombre
FROM departamentos d
WHERE EXISTS (
    SELECT 1
    FROM usuarios u
    WHERE u.departamento_id = d.id AND u.estado = 'activo'
);

-- Departamentos sin usuarios activos
SELECT
    d.id,
    d.nombre
FROM departamentos d
WHERE NOT EXISTS (
    SELECT 1
    FROM usuarios u
    WHERE u.departamento_id = d.id AND u.estado = 'activo'
);
```

---

## NIVEL 7: Características Avanzadas

### 7.1 - UNION / UNION ALL / INTERSECT / EXCEPT
```sql
-- Combinar resultados de dos consultas
SELECT nombre, 'Cliente' AS tipo FROM clientes
UNION
SELECT nombre, 'Proveedor' AS tipo FROM proveedores
ORDER BY nombre;

-- UNION ALL mantiene duplicados
SELECT email FROM usuarios_antiguos
UNION ALL
SELECT email FROM usuarios_nuevos;

-- INTERSECT: solo registros en ambas tablas
SELECT id FROM usuarios_2023
INTERSECT
SELECT id FROM usuarios_2024;

-- EXCEPT: en la primera pero no en la segunda
SELECT id FROM usuarios_2023
EXCEPT
SELECT id FROM usuarios_2024;
```

### 7.2 - CASE avanzado con agregación
```sql
-- Dashboard de usuarios por categoría
SELECT
    DATE_TRUNC('month', fecha_registro) AS mes,
    COUNT(CASE WHEN edad < 30 THEN 1 END) AS usuarios_jovenes,
    COUNT(CASE WHEN edad BETWEEN 30 AND 60 THEN 1 END) AS usuarios_medios,
    COUNT(CASE WHEN edad > 60 THEN 1 END) AS usuarios_mayores,
    COUNT(CASE WHEN ingresos > 100000 THEN 1 END) AS usuarios_altos_ingresos,
    COUNT(*) AS total
FROM usuarios
GROUP BY DATE_TRUNC('month', fecha_registro)
ORDER BY mes DESC;
```

### 7.3 - Recursivas (PostgreSQL) - CTEs recursivas
```sql
-- Tabla jerárquica de empleados
-- Tabla: empleados (id, nombre, gerente_id, departamento)

WITH RECURSIVE arbol_empleados AS (
    -- Caso base: empleados sin gerente (directores)
    SELECT
        id,
        nombre,
        gerente_id,
        departamento,
        1 AS nivel,
        nombre AS ruta
    FROM empleados
    WHERE gerente_id IS NULL

    UNION ALL

    -- Caso recursivo: subordinados
    SELECT
        e.id,
        e.nombre,
        e.gerente_id,
        e.departamento,
        ae.nivel + 1,
        ae.ruta || ' -> ' || e.nombre
    FROM empleados e
    INNER JOIN arbol_empleados ae ON e.gerente_id = ae.id
    WHERE ae.nivel < 10
)
SELECT * FROM arbol_empleados ORDER BY ruta;
```

### 7.4 - Arrays (PostgreSQL específico)
```sql
-- Trabajar con arrays
SELECT
    nombre,
    skills,
    ARRAY_LENGTH(skills, 1) AS cantidad_skills,
    skills[1] AS primer_skill
FROM usuarios
WHERE 'Python' = ANY(skills);

-- Agrupar en array
SELECT
    departamento_id,
    ARRAY_AGG(nombre) AS usuarios,
    COUNT(*) AS total
FROM usuarios
GROUP BY departamento_id;
```

### 7.5 - JSON (PostgreSQL y BigQuery)
```sql
-- PostgreSQL
SELECT
    nombre,
    metadata,
    metadata->>'edad' AS edad,
    metadata->>'ciudad' AS ciudad,
    (metadata->>'ingresos')::NUMERIC AS ingresos
FROM usuarios
WHERE metadata ? 'edad';

-- BigQuery
SELECT
    nombre,
    JSON_EXTRACT_SCALAR(metadata, '$.edad') AS edad,
    CAST(JSON_EXTRACT_SCALAR(metadata, '$.ingresos') AS FLOAT64) AS ingresos
FROM usuarios;
```

### 7.6 - Window functions avanzadas (streaming)
```sql
-- Análisis de tendencia: comparar semana actual vs semana anterior
WITH ventas_semanales AS (
    SELECT
        DATE_TRUNC('week', fecha) AS semana,
        SUM(monto) AS total_semana,
        COUNT(*) AS transacciones
    FROM transacciones
    GROUP BY DATE_TRUNC('week', fecha)
)
SELECT
    semana,
    total_semana,
    LAG(total_semana) OVER (ORDER BY semana) AS semana_anterior,
    total_semana - LAG(total_semana) OVER (ORDER BY semana) AS cambio_absoluto,
    ROUND(((total_semana - LAG(total_semana) OVER (ORDER BY semana))
        / LAG(total_semana) OVER (ORDER BY semana) * 100), 2) AS cambio_porcentaje
FROM ventas_semanales
ORDER BY semana DESC;
```

### 7.7 - Materialización de fechas (BigQuery - tabla de dimensión)
```sql
-- Útil para filtros rápidos
CREATE OR REPLACE TABLE project.dataset.dim_fechas AS
WITH fecha_base AS (
    SELECT DATE('2020-01-01') + GENERATE_ARRAY(0, 3650)[OFFSET(0)] as fecha
)
SELECT
    fecha,
    EXTRACT(YEAR FROM fecha) AS año,
    EXTRACT(MONTH FROM fecha) AS mes,
    EXTRACT(QUARTER FROM fecha) AS trimestre,
    FORMAT_DATE('%A', fecha) AS dia_semana,
    EXTRACT(WEEK FROM fecha) AS semana_del_año
FROM fecha_base;
```

### 7.8 - Análisis de cohortes (avanzado)
```sql
-- Rastrear retencion de usuarios por mes de registro
WITH
cohort_data AS (
    SELECT
        u.id,
        DATE_TRUNC('month', u.fecha_registro) AS cohort_month,
        DATE_TRUNC('month', t.fecha_transaccion) AS transaction_month,
        DATE_PART('month', t.fecha_transaccion - u.fecha_registro) AS meses_desde_registro
    FROM usuarios u
    LEFT JOIN transacciones t ON u.id = t.usuario_id
),
cohort_sizes AS (
    SELECT
        cohort_month,
        COUNT(DISTINCT id) AS cohort_size
    FROM cohort_data
    WHERE transaction_month IS NOT NULL
    GROUP BY cohort_month
)
SELECT
    cd.cohort_month,
    CAST(cd.meses_desde_registro AS INT) AS meses,
    COUNT(DISTINCT cd.id) AS usuarios_activos,
    ROUND(COUNT(DISTINCT cd.id) * 100.0 / cs.cohort_size, 2) AS retencion_porcentaje
FROM cohort_data cd
LEFT JOIN cohort_sizes cs ON cd.cohort_month = cs.cohort_month
WHERE cd.transaction_month IS NOT NULL
GROUP BY cd.cohort_month, meses, cs.cohort_size
ORDER BY cohort_month, meses;
```

---

## Ejemplos Complejos Integradores

### Ejemplo 1: Dashboard de RR.HH.
```sql
WITH
-- Datos base
empleados_info AS (
    SELECT
        e.id,
        e.nombre,
        d.nombre AS departamento,
        e.salario,
        DATE_PART('year', AGE(e.fecha_nacimiento)) AS edad,
        DATE_PART('year', AGE(e.fecha_ingreso)) AS años_empresa,
        e.estado
    FROM empleados e
    LEFT JOIN departamentos d ON e.departamento_id = d.id
),
-- Ranking dentro del departamento
ranking_salarios AS (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY departamento ORDER BY salario DESC) AS ranking_dept,
        AVG(salario) OVER (PARTITION BY departamento) AS salario_promedio_dept
    FROM empleados_info
)
SELECT
    nombre,
    departamento,
    salario,
    ranking_dept,
    salario_promedio_dept,
    ROUND(salario - salario_promedio_dept, 2) AS diferencia_promedio,
    CASE
        WHEN ranking_dept <= 3 THEN 'Top 3'
        WHEN salario > salario_promedio_dept THEN 'Superior promedio'
        ELSE 'Inferior promedio'
    END AS categoria_salario,
    edad,
    años_empresa
FROM ranking_salarios
WHERE estado = 'activo'
ORDER BY departamento, ranking_dept;
```

### Ejemplo 2: Análisis de ventas con tendencias
```sql
WITH ventas_diarias AS (
    SELECT
        DATE_TRUNC('day', fecha) AS dia,
        vendedor_id,
        SUM(monto) AS total_dia,
        COUNT(*) AS transacciones
    FROM transacciones
    GROUP BY DATE_TRUNC('day', fecha), vendedor_id
),
tendencias AS (
    SELECT
        dia,
        vendedor_id,
        total_dia,
        transacciones,
        AVG(total_dia) OVER (
            PARTITION BY vendedor_id
            ORDER BY dia
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS promedio_7_dias,
        LAG(total_dia) OVER (
            PARTITION BY vendedor_id
            ORDER BY dia
        ) AS dia_anterior,
        ROUND(((total_dia - LAG(total_dia) OVER (PARTITION BY vendedor_id ORDER BY dia))
            / LAG(total_dia) OVER (PARTITION BY vendedor_id ORDER BY dia) * 100), 2) AS cambio_porcentaje
    FROM ventas_diarias
)
SELECT
    v.nombre AS vendedor,
    t.dia,
    t.total_dia,
    t.promedio_7_dias,
    t.cambio_porcentaje,
    CASE
        WHEN t.cambio_porcentaje > 10 THEN 'Crecimiento fuerte'
        WHEN t.cambio_porcentaje > 0 THEN 'Crecimiento'
        WHEN t.cambio_porcentaje > -10 THEN 'Caída leve'
        ELSE 'Caída fuerte'
    END AS tendencia
FROM tendencias t
JOIN vendedores v ON t.vendedor_id = v.id
WHERE t.dia >= CURRENT_DATE - INTERVAL '3 months'
ORDER BY v.nombre, t.dia DESC;
```

### Ejemplo 3: Segmentación de clientes (RFM)
```sql
-- RFM: Recency, Frequency, Monetary
WITH transacciones AS (
    SELECT
        cliente_id,
        MAX(fecha) AS ultima_compra,
        COUNT(*) AS frecuencia,
        SUM(monto) AS valor_total_compras
    FROM transacciones
    GROUP BY cliente_id
),
rfm AS (
    SELECT
        cliente_id,
        DATE_PART('day', CURRENT_DATE - ultima_compra) AS dias_sin_comprar,
        frecuencia,
        valor_total_compras,
        NTILE(5) OVER (ORDER BY DATE_PART('day', CURRENT_DATE - ultima_compra) DESC) AS recency_tier,
        NTILE(5) OVER (ORDER BY frecuencia) AS frequency_tier,
        NTILE(5) OVER (ORDER BY valor_total_compras) AS monetary_tier
    FROM transacciones
)
SELECT
    c.nombre,
    rfm.dias_sin_comprar,
    rfm.frecuencia,
    rfm.valor_total_compras,
    CASE
        WHEN recency_tier >= 4 AND frequency_tier >= 4 AND monetary_tier >= 4 THEN 'Champions'
        WHEN recency_tier >= 3 AND frequency_tier >= 4 AND monetary_tier >= 4 THEN 'Fieles'
        WHEN recency_tier >= 3 AND frequency_tier >= 3 AND monetary_tier >= 3 THEN 'Potencial'
        WHEN recency_tier <= 2 AND frequency_tier >= 4 THEN 'En riesgo'
        WHEN recency_tier <= 1 THEN 'Dormidos'
        ELSE 'Otros'
    END AS segmento
FROM rfm
JOIN clientes c ON rfm.cliente_id = c.id
ORDER BY segmento, rfm.valor_total_compras DESC;
```

---

## Diferencias entre PostgreSQL y BigQuery

### PostgreSQL (SQL tradicional)
```
✓ Funciones de ventana nativas
✓ CTEs recursivas
✓ Arrays y JSON
✓ Expresiones regulares nativas (regex)
✓ Transacciones ACID
✓ Índices avanzados
✗ Menos escalable para big data
```

### BigQuery (SQL para analytics)
```
✓ Muy escalable para millones de filas
✓ Análisis rápido de datos masivos
✓ Integración con Google Cloud
✓ STRUCT y ARRAY como tipos
✓ Funciones especializadas de análisis
✗ No es una base de datos transaccional
✗ Menos funciones nativas (aunque mejorando)
```

### Sintaxis diferente
```sql
-- PostgreSQL
DATE_TRUNC('month', fecha)
SUBSTRING(nombre, 1, 5)
ARRAY_AGG(valor)

-- BigQuery
DATE_TRUNC(fecha, MONTH)
SUBSTR(nombre, 1, 5)
ARRAY_AGG(valor)
```

---

## Ejercicios Prácticos

### Ejercicio 1: Principiante
```sql
-- Obtener el nombre, edad y ciudad de todos los usuarios
-- mayores de 25 años, ordenados alfabéticamente
-- Respuesta en: seccion siguiente
```

### Ejercicio 2: Intermedio
```sql
-- Contar cuántas transacciones tuvo cada cliente el año pasado,
-- mostrando solo a aquellos con más de 5 transacciones,
-- ordenado por cantidad descendente
```

### Ejercicio 3: Avanzado
```sql
-- Crear un ranking de productos dentro de cada categoría
-- mostrando: nombre, categoría, precio, cantidad vendida,
-- ingresos totales, ranking en categoría y porcentaje del total de la categoría
```

### Ejercicio 4: Experto
```sql
-- Análisis de churn: Identificar clientes que compraron en 2024
-- pero no en 2025, mostrando su histórico de compras,
-- frecuencia de compra, últimas 3 fechas de compra y
-- estimado de su valor de vida
```

---

## Consejos de Optimización

1. **Usa índices** en columnas WHERE frecuentes
2. **Evita SELECT *** en tablas grandes
3. **Agrupa datos** antes de hacer joins
4. **Usa EXPLAIN** para analizar planes de ejecución
5. **Las CTEs son tus amigas** para legibilidad
6. **Window functions > Subconsultas** en performance
7. **LIMIT temprano** en desarrollo
8. **Comenta código complejo**

---

¡Ahora estás listo para escribir SQL profesional!
