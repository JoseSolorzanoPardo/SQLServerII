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








# Modificación de estructura de tablas

Sea la siguiente tabla de Usuarios:

**🧾 Tabla: Usuarios**
```
CREATE TABLE Usuarios (
id_usuario INT PRIMARY KEY,
nombre_completo VARCHAR(100) NOT NULL,
correo VARCHAR(100) UNIQUE, -- Puede ser NULL
fecha_registro DATE,
nivel_ingresos VARCHAR(20), -- 'Alto', 'Medio', 'Bajo'
segmento_cliente VARCHAR(30) -- 'Premium', 'Frecuente', 'Ocasional'
);
```


**Campos clave explicados:**

- id_usuario: Clave primaria.
- nombre_completo: Nombre del usuario.
- correo: Para contacto o marketing, único por usuario.
- fecha_registro: Para saber cuándo entró al sistema.
- nivel_ingresos: Aporta al análisis de riesgo financiero.
- segmento_cliente: Útil para campañas y estrategias comerciales.

**Registros para poblar la tabla anterior**

```
INSERT INTO Usuarios (id_usuario, nombre_completo, correo, fecha_registro, nivel_ingresos, segmento_cliente) VALUES
(1, 'Ana María Torres', 'ana.torres@email.com', '2024-01-10', 'Alto', 'Premium'),
(2, 'Carlos Gómez', 'carlos.gomez@email.com', '2024-02-15', 'Medio', 'Frecuente'),
(3, 'Laura Rodríguez', 'laura.rodriguez@email.com', '2024-03-05', 'Bajo', 'Ocasional'),
(4, 'Javier Fernández', 'javier.fernandez@email.com', '2023-12-20', 'Medio', 'Frecuente'),
(5, 'Diana Morales', 'diana.morales@email.com', '2024-01-25', 'Alto', 'Premium'),
(6, 'Andrés Pérez', 'andres.perez@email.com', '2023-11-30', 'Bajo', 'Ocasional'),
(7, 'Luisa Herrera', 'luisa.herrera@email.com', '2024-03-01', 'Medio', 'Frecuente'),
(8, 'Tomás Ramírez', 'tomas.ramirez@email.com', '2024-02-10', 'Alto', 'Premium'),
(9, 'Paula Castillo', 'paula.castillo@email.com', '2024-04-02', 'Medio', 'Frecuente'),
(10, 'Esteban Ruiz', 'esteban.ruiz@email.com', '2024-01-17', 'Bajo', 'Ocasional'),
(11, 'Camila Pardo', 'camila.pardo@email.com', '2024-01-09', 'Alto', 'Frecuente'),
(12, 'Diego Martínez', 'diego.martinez@email.com', '2024-02-12', 'Medio', 'Ocasional'),
(13, 'Marcela Vargas', 'marcela.vargas@email.com', '2024-03-22', 'Bajo', 'Ocasional'),
(14, 'Ricardo Cárdenas', 'ricardo.cardenas@email.com', '2024-03-03', 'Alto', 'Premium'),
(15, 'Tatiana Salazar', 'tatiana.salazar@email.com', '2024-02-05', 'Medio', 'Frecuente'),
(16, 'Miguel Lozano', 'miguel.lozano@email.com', '2023-12-10', 'Bajo', 'Ocasional'),
(17, 'Valentina Ortiz', 'valentina.ortiz@email.com', '2024-04-10', 'Medio', 'Frecuente'),
(18, 'José Ángel Soto', 'jose.soto@email.com', '2024-03-15', 'Alto', 'Premium'),
(19, 'Natalia Mendoza', 'natalia.mendoza@email.com', '2024-01-28', 'Medio', 'Frecuente'),
(20, 'Felipe Ríos', 'felipe.rios@email.com', '2024-02-20', 'Bajo', 'Ocasional');
```
Con la siguiente consulta podemos determinar o conocer la estructura de una tabla en particular de nuestra base de datos:
```
SELECT
COLUMN_NAME,
DATA_TYPE,
CHARACTER_MAXIMUM_LENGTH,
IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Usuarios';
```

**1\. Agregar una nueva columna: telefono**
```
ALTER TABLE Usuarios
ADD telefono VARCHAR(15);
```

_Contexto pedagógico:_ se detectó que la empresa quiere hacer campañas por WhatsApp.

**Eliminar el campo agregado:**
```
ALTER TABLE Usuarios
DROP COLUMN telefono;
```

**2\. Modificar el tipo de dato de una columna: ampliar correo**
```
ALTER TABLE Usuarios
ALTER COLUMN correo VARCHAR(150);
```

_Contexto pedagógico:_ los correos institucionales exceden los 100 caracteres.

**3\. Agregar una restricción CHECK: para validar nivel_ingresos**
```
ALTER TABLE Usuarios
ADD CONSTRAINT chk_nivel_ingresos
CHECK (nivel_ingresos IN ('Alto', 'Medio', 'Bajo'));
```
_Contexto pedagógico:_ se asegura integridad en el ingreso de datos válidos.

**Ejemplo para verificar restricción:**
```
INSERT INTO Usuarios (id_usuario, nombre_completo, correo, fecha_registro, nivel_ingresos, segmento_cliente)
VALUES (21, 'Luis Barreto', 'luis.barreto@email.com', '2024-05-11', 'Muy Alto', 'Premium');
```
**Consulta para ver las restricciones de un campo:**
```
SELECT
con.name AS nombre_restriccion,
col.name AS columna_afectada,
con.definition AS regla
FROM sys.check_constraints con
JOIN sys.columns col ON con.parent_object_id = col.object_id AND con.parent_column_id = col.column_id
WHERE OBJECT_NAME(con.parent_object_id) = 'Usuarios';
```

**4\. Agregar una clave foránea (FOREIGN KEY): relacionar con una tabla Ciudades**

-- Primero supongamos que existe una tabla Ciudades
```
CREATE TABLE Ciudades (
id_ciudad INT PRIMARY KEY,
nombre_ciudad VARCHAR(100)
);
```

```
INSERT INTO Ciudades (id_ciudad, nombre_ciudad) VALUES
(1, 'Bogotá'),
(2, 'Medellín'),
(3, 'Cali'),
(4, 'Barranquilla'),
(5, 'Cartagena'),
(6, 'Manizales'),
(7, 'Pereira'),
(8, 'Bucaramanga'),
(9, 'Cúcuta'),
(10, 'Santa Marta');
```

-- Luego agregamos la columna y la clave foránea
```
ALTER TABLE Usuarios
ADD id_ciudad INT;
ALTER TABLE Usuarios
ADD CONSTRAINT fk_ciudad_usuario
FOREIGN KEY (id_ciudad) REFERENCES Ciudades(id_ciudad);
```

_Contexto pedagógico:_ ejemplo típico de normalización y relaciones entre tablas.

Pasamos agregar los registros correspondientes en la tabla:
```
INSERT INTO Usuarios (id_usuario, nombre_completo, correo, fecha_registro, nivel_ingresos, segmento_cliente, id_ciudad) VALUES
(22, 'Sebastián Quintero', 'sebastian.quintero@email.com', '2024-04-16', 'Medio', 'Frecuente', 2),
(23, 'Valeria Meza', 'valeria.meza@email.com', '2024-04-17', 'Bajo', 'Ocasional', 3),
(24, 'Mauricio Díaz', 'mauricio.diaz@email.com', '2024-04-18', 'Medio', 'Frecuente', 4),
(25, 'Alejandra Patiño', 'alejandra.patino@email.com', '2024-04-19', 'Alto', 'Premium', 5),
(26, 'Cristian Rojas', 'cristian.rojas@email.com', '2024-04-20', 'Bajo', 'Ocasional', 6),
(27, 'Mónica Benítez', 'monica.benitez@email.com', '2024-04-21', 'Medio', 'Frecuente', 7),
(28, 'Daniel Ortega', 'daniel.ortega@email.com', '2024-04-22', 'Alto', 'Premium', 8),
(29, 'Natalia Ramírez', 'natalia.ramirez@email.com', '2024-04-23', 'Medio', 'Frecuente', 9),
(30, 'Jhonatan Silva', 'jhonatan.silva@email.com', '2024-04-24', 'Bajo', 'Ocasional', 10);
```

**5\. Eliminar una columna: segmento_cliente**
```
ALTER TABLE Usuarios
DROP COLUMN segmento_cliente;
```

_Contexto pedagógico:_ simula una decisión de negocio donde ese campo ya no se usa.
