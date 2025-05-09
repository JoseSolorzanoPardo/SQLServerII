# ¿Qué es una subconsulta?

Una subconsulta es una consulta anidada dentro de otra consulta, que se ejecuta primero y su resultado se usa en la consulta principal.

## Mostrar todas las transacciones que tienen el monto más alto

Primero, se puede sacar el monto más alto:
```
SELECT MAX(monto) FROM TransaccionesFraude;
```
Ahora vamos a usar esa subconsulta dentro de una consulta principal para obtener la fila completa que tiene ese monto:

**Subconsulta en cláusula WHERE:**
```
SELECT *
FROM TransaccionesFraude
WHERE monto = (SELECT MAX(monto) FROM TransaccionesFraude);
```
Esta consulta retorna la(s) fila(s) completa(s) con el monto más alto.

**2\. Subconsulta con IN (múltiples valores)**

**Mostrar transacciones hechas desde países que tienen más de 2 transacciones**
```
SELECT *
FROM TransaccionesFraude
WHERE pais IN (
SELECT pais
FROM TransaccionesFraude
GROUP BY pais
HAVING COUNT(*) > 2
);
```
**3\. Subconsulta en FROM (como tabla derivada)**

**Obtener el promedio por país, y filtrar los que tienen más de $1.000.000**
```
SELECT pais, Promedio
FROM (
SELECT pais, AVG(monto) AS Promedio
FROM TransaccionesFraude
GROUP BY pais
) AS tabla_promedios
WHERE Promedio > 1000000;
```
**4\. Subconsulta en SELECT (columna calculada)**

**Para cada transacción, mostrar el promedio general como una columna adicional**
```
SELECT
id_transaccion,
monto,
(SELECT AVG(monto) FROM TransaccionesFraude) AS Promedio_General
FROM TransaccionesFraude;
```

**5\. Subconsulta en HAVING**

**Mostrar países cuyo total supera el total de Colombia**
```
SELECT pais, SUM(monto) AS Total
FROM TransaccionesFraude
GROUP BY pais
HAVING SUM(monto) > (
SELECT SUM(monto)
FROM TransaccionesFraude
WHERE pais = 'Colombia'
);
```

# Estructura general de CASE WHEN
```
SELECT
columna,
CASE
WHEN condición1 THEN resultado1
WHEN condición2 THEN resultado2
...
ELSE resultado_por_defecto
END AS nombre_columna_resultado
FROM tabla;
```

- **WHEN** evalúa una condición.
- **THEN** define qué valor devolver si esa condición se cumple.
- **ELSE** (opcional) es el valor que se devuelve si **ninguna condición anterior** se cumple.
- **END** cierra la sentencia CASE.

**Ejemplo con tabla TransaccionesFraude**

Supón que quieres categorizar las transacciones por **valor monetario**:
```
SELECT
id_transaccion,
monto,
CASE
WHEN monto >= 1500000 THEN 'Muy Alta'
WHEN monto >= 800000 THEN 'Alta'
WHEN monto >= 400000 THEN 'Media'
ELSE 'Baja'
END AS Categoria_Monto
FROM TransaccionesFraude;
```
**¿Qué hace esto?**

- Si monto ≥ 1.500.000 → "Muy Alta"
- Si monto ≥ 800.000 → "Alta"
- Si monto ≥ 400.000 → "Media"
- De lo contrario → "Baja"

## Ejemplo

Queremos generar una columna llamada Accion_Sugerida, que nos diga qué hacer según el riesgo y el resultado de validación.

**Consulta con CASE:**
```
SELECT
id_transaccion,
resultado_validacion,
riesgo_fraude,
monto,
CASE
WHEN resultado_validacion = 'Rechazado' AND riesgo_fraude = 'Alto' THEN 'Bloquear usuario y escalar'
WHEN resultado_validacion = 'Pendiente' AND riesgo_fraude = 'Alto' THEN 'Revisión manual urgente'
WHEN resultado_validacion = 'Aprobado' AND riesgo_fraude = 'Medio' THEN 'Monitorear'
WHEN resultado_validacion = 'Aprobado' AND riesgo_fraude = 'Bajo' THEN 'Sin acción'
ELSE 'Revisión estándar'
END AS Accion_Sugerida
FROM TransaccionesFraude;
```
**¿Qué hace esta lógica?**

- Si fue **rechazada** y el riesgo es **alto** → actuar con urgencia.
- Si es **pendiente** y el riesgo es **alto** → revisión manual.
- Si fue **aprobada**, se actúa según el nivel de riesgo:
  - Medio → monitorear
  - Bajo → sin acción
- En cualquier otro caso → revisión estándar.

# ¿Qué es una variable?

Es un contenedor con nombre que puede guardar un valor temporal (número, texto, fecha, etc.) durante la ejecución de un script SQL.

**1\. Declaración de una variable**
```
DECLARE @nombre_variable TIPO_DE_DATO;
```
**Ejemplos:**
```
DECLARE @nombre VARCHAR(100); -- texto
DECLARE @edad INT; -- número entero
DECLARE @fecha_actual DATE; -- fecha
DECLARE @total DECIMAL(10, 2); -- número decimal
```

**2\. Asignación de valor**

**Opción A: con SET**
```
SET @nombre = 'Jose';
SET @edad = 30;
```
**Opción B: con SELECT**
```
SELECT @total = SUM(monto) FROM TransaccionesFraude;
SELECT es útil cuando asignas **valores desde una consulta**.
```
**3\. Uso de variables**

Puedes usarlas en sentencias como IF, PRINT, INSERT, UPDATE, etc.
```
IF @edad > 18
PRINT 'Es mayor de edad';
```
**Ejemplo completo:**
```
DECLARE @totalTransacciones INT;
DECLARE @promedioMonto DECIMAL(10,2);

SELECT @totalTransacciones = COUNT(*),
@promedioMonto = AVG(monto)
FROM TransaccionesFraude;

PRINT 'Total de transacciones: ' + CAST(@totalTransacciones AS VARCHAR);
PRINT 'Promedio de monto: ' + CAST(@promedioMonto AS VARCHAR);
```
**Tipos de datos comunes para variables**

| **Tipo** | **Descripción** |
| --- | --- |
| INT | Entero |
| DECIMAL | Números con decimales |
| VARCHAR(n) | Texto variable (n caracteres) |
| DATE | Fecha |
| DATETIME | Fecha y hora |
| BIT | Booleano (0 o 1) |
