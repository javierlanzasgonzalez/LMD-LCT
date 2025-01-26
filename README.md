# **MySQL (LMD & LCT)**

## **1. Lenguaje de Manipulación de Datos (LMD)**

### **1.1 Inserción de Registros**

- **Sintaxis Básica**:
  ```sql
  INSERT INTO tabla (columna1, columna2) VALUES (valor1, valor2);
  ```
- **Insertar múltiples filas**:
  ```sql
  INSERT INTO tabla (columna1, columna2) VALUES
  (valor1, valor2),
  (valor3, valor4);
  ```
- **Desde otra tabla**:
  ```sql
  INSERT INTO nueva_tabla (columna1, columna2)
  SELECT columna1, columna2
  FROM tabla_original
  WHERE condicion;
  ```

### **1.2 Modificación de Registros**

- **Actualizar registros**:
  ```sql
  UPDATE tabla
  SET columna1 = nuevo_valor1, columna2 = nuevo_valor2
  WHERE condicion;
  ```
- **Actualizar con subconsultas**:
  ```sql
  UPDATE empleados
  SET salario = salario + 500
  WHERE id_departamento IN (
      SELECT id FROM departamentos WHERE nombre = 'Ventas'
  );
  ```

### **1.3 Eliminación de Registros**

- **Eliminar registros**:
  ```sql
  DELETE FROM tabla WHERE condicion;
  ```
- **Eliminar todos los registros** (Usar con cuidado):
  ```sql
  DELETE FROM tabla;
  ```
- **Con subconsultas**:
  ```sql
  DELETE FROM empleados
  WHERE id NOT IN (
      SELECT id_empleado FROM proyectos
  );
  ```

---

## **2. Lenguaje de Control de Transacciones (LCT)**

### **2.1 Transacciones Básicas**

- **Iniciar una transacción**:
  ```sql
  START TRANSACTION;
  ```
- **Confirmar cambios**:
  ```sql
  COMMIT;
  ```
- **Deshacer cambios**:
  ```sql
  ROLLBACK;
  ```

### **2.2 Aislamiento de Transacciones**

- **Establecer nivel de aislamiento**:
  ```sql
  SET TRANSACTION ISOLATION LEVEL nivel;
  ```
  Niveles disponibles:
  - `READ UNCOMMITTED`: Lectura de datos no confirmados.
  - `READ COMMITTED`: Lectura de datos confirmados.
  - `REPEATABLE READ`: Previene lecturas no repetibles.
  - `SERIALIZABLE`: Máxima consistencia.

  # Niveles de Aislamiento y Problemas

| **Nivel de Aislamiento** | **Problemas Comunes**                             | **Descripción del Problema**                                                                                       | **Soluciones**                                                                                                   |
|---------------------------|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| READ UNCOMMITTED          | Lecturas sucias, lecturas no repetibles, phantoms | Permite leer datos no confirmados que pueden ser revertidos, causando inconsistencias.                             | Utilizar niveles de aislamiento más estrictos como READ COMMITTED o superiores.                                 |
| READ COMMITTED            | Lecturas no repetibles, phantoms                 | Evita lecturas sucias, pero un mismo registro puede ser leído con valores diferentes durante la transacción.       | Utilizar REPEATABLE READ para prevenir cambios en los datos ya leídos.                                         |
| REPEATABLE READ           | Phantoms                                        | Evita lecturas sucias y no repetibles, pero no previene la aparición de nuevos registros visibles para la transacción. | Implementar SERIALIZABLE para evitar inserciones o modificaciones externas durante la transacción.             |
| SERIALIZABLE              | Ningún problema                                 | Proporciona máxima consistencia bloqueando toda la tabla para evitar cualquier interferencia.                      | No requiere soluciones, pero reduce la concurrencia y el rendimiento.                                          |

---

## Aquí tienes ejemplos prácticos para cada nivel de aislamiento en MySQL, con escenarios reales que muestran cómo funcionan y qué problemas se pueden evitar o permitir:

---

### **1. READ UNCOMMITTED**
Permite leer datos no confirmados, lo que puede causar **lecturas sucias**.

#### **Ejemplo**:
- **Transacción 1**:
  ```sql
  START TRANSACTION;
  UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
  ```
- **Transacción 2** (antes de que la primera haga COMMIT o ROLLBACK):
  ```sql
  SELECT saldo FROM cuentas WHERE id = 1;
  ```
- **Resultado**:
  Transacción 2 puede ver el cambio temporal (-100 en el saldo) aunque Transacción 1 decida hacer `ROLLBACK`.

---

### **2. READ COMMITTED**
Evita **lecturas sucias**, pero permite **lecturas no repetibles**.

#### **Ejemplo**:
- **Transacción 1**:
  ```sql
  START TRANSACTION;
  UPDATE productos SET precio = 20 WHERE id = 1;
  COMMIT;
  ```
- **Transacción 2**:
  ```sql
  START TRANSACTION;
  SELECT precio FROM productos WHERE id = 1; -- Resultado: 20
  UPDATE productos SET precio = 25 WHERE id = 1;
  COMMIT;
  SELECT precio FROM productos WHERE id = 1; -- Resultado: 25
  ```
- **Resultado**:
  El valor leído por Transacción 2 puede cambiar entre dos lecturas, mostrando **lecturas no repetibles**.

---

### **3. REPEATABLE READ**
Evita **lecturas sucias** y **lecturas no repetibles**, pero no evita **phantoms** (filas que aparecen inesperadamente).

#### **Ejemplo**:
- **Transacción 1**:
  ```sql
  START TRANSACTION;
  SELECT * FROM pedidos WHERE cliente_id = 1; -- Devuelve 5 pedidos
  ```
- **Transacción 2**:
  ```sql
  START TRANSACTION;
  INSERT INTO pedidos (cliente_id, producto_id) VALUES (1, 101);
  COMMIT;
  ```
- **Transacción 1 (de nuevo)**:
  ```sql
  SELECT * FROM pedidos WHERE cliente_id = 1; -- Devuelve los mismos 5 pedidos (sin incluir el nuevo).
  COMMIT;
  ```
- **Resultado**:
  Transacción 1 no ve los cambios realizados por Transacción 2 durante la transacción activa, evitando **lecturas no repetibles**. Sin embargo, si Transacción 1 consulta un rango mayor después de COMMIT, podría aparecer un "phantom".

---

### **4. SERIALIZABLE**
Proporciona máxima consistencia, bloqueando las filas afectadas por cualquier operación, lo que evita **lecturas sucias**, **lecturas no repetibles** y **phantoms**.

#### **Ejemplo**:
- **Transacción 1**:
  ```sql
  SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  START TRANSACTION;
  SELECT * FROM ventas WHERE producto_id = 1; -- Devuelve 10 ventas
  ```
- **Transacción 2**:
  ```sql
  SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  START TRANSACTION;
  INSERT INTO ventas (producto_id, cliente_id) VALUES (1, 5);
  -- Esto queda bloqueado hasta que Transacción 1 haga COMMIT o ROLLBACK.
  ```
- **Resultado**:
  Transacción 2 no puede realizar cambios mientras Transacción 1 esté activa, evitando cualquier inconsistencia.

---

### **Resumen**
| **Nivel de Aislamiento** | **Problemas Evitados**                  | **Problemas Permitidos**         |
|---------------------------|----------------------------------------|----------------------------------|
| READ UNCOMMITTED          | Ninguno                               | Lecturas sucias, no repetibles, phantoms |
| READ COMMITTED            | Lecturas sucias                      | Lecturas no repetibles, phantoms |
| REPEATABLE READ           | Lecturas sucias, no repetibles        | Phantoms                        |
| SERIALIZABLE              | Lecturas sucias, no repetibles, phantoms | Ninguno                         |

Estos ejemplos te ayudan a visualizar cómo afectan los niveles de aislamiento a las transacciones y a elegir el más adecuado para tu caso. 😊

### **2.3 Políticas de Bloqueo**

- **Bloquear registros para actualización**:
  ```sql
  SELECT * FROM tabla WHERE condicion FOR UPDATE;
  ```

---

## **3. Integridad y Consistencia**

### **3.1 Claves Foráneas**

- **Crear clave foránea**:
  ```sql
  ALTER TABLE tabla_hija
  ADD CONSTRAINT fk_nombre FOREIGN KEY (columna_hija)
  REFERENCES tabla_padre(columna_padre)
  ON DELETE CASCADE
  ON UPDATE CASCADE;
  ```

### **3.2 Políticas de Eliminación**

- **ON DELETE CASCADE**: Elimina registros hijos al eliminar el padre.
- **ON DELETE SET NULL**: Establece `NULL` en registros hijos.
- **ON DELETE RESTRICT**: Previene eliminaciones si hay referencias.

| **Cláusula**      | **Comportamiento en DELETE/UPDATE**                                                               | **Ejemplo de Comando**                                                                                               |
|--------------------|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| **`SET NULL`**     | Establece `NULL` en las columnas hijas relacionadas.                                             | `FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE SET NULL ON UPDATE SET NULL`                             |
| **`SET DEFAULT`**  | Establece un valor por defecto en las columnas hijas relacionadas (no soportado directamente).    | Implementar mediante un **trigger** (ver ejemplo): `CREATE TRIGGER ... UPDATE pedidos SET cliente_id = 0 ...`        |
| **`CASCADE`**      | Elimina o actualiza automáticamente los registros relacionados.                                   | `FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE CASCADE ON UPDATE CASCADE`                               |
| **`RESTRICT`**     | Impide eliminar o actualizar registros si existen relaciones.                                     | `FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE RESTRICT ON UPDATE RESTRICT`                             |
| **`NO ACTION`**    | Similar a `RESTRICT`, pero la verificación ocurre al final de la transacción.                     | `FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE NO ACTION ON UPDATE NO ACTION`                           |


### **3.3 Validación de Datos**

- **Restricciones comunes**:
  - `NOT NULL`: Impide valores nulos.
  - `UNIQUE`: Garantiza valores únicos.
  - `CHECK`: Valida condiciones personalizadas.

### **3.4 Ejemplo Complejo de Integridad**

- **Actualizar en cascada**:
  ```sql
  UPDATE departamentos
  SET id = '42'
  WHERE id = '02';
  ```
  Esto actualizará automáticamente las claves foráneas en las tablas relacionadas.

---

## **4. Diseño de Consultas Avanzadas**

### **4.1 Uso de Subconsultas en LMD**

- **Insertar con subconsulta**:
  ```sql
  INSERT INTO empleados_backup (id, nombre)
  SELECT id, nombre FROM empleados WHERE salario > 3000;
  ```
- **Actualizar con subconsulta**:
  ```sql
  UPDATE empleados
  SET salario = salario * 1.1
  WHERE id IN (
      SELECT id FROM empleados WHERE antigüedad > 5
  );
  ```

### **4.2 Creación de Tablas Temporales**

- **Tabla temporal para operaciones complejas**:
  ```sql
  CREATE TEMPORARY TABLE empleados_temp AS
  SELECT * FROM empleados WHERE salario > 3000;
  ```
  Estas tablas se eliminan al finalizar la sesión.

---

## **5. Buenas Prácticas**

1. **Habilitar y verificar transacciones**: Siempre utiliza `COMMIT` o `ROLLBACK` según sea necesario.
2. **Respetar integridad referencial**: Utiliza claves foráneas para mantener relaciones consistentes.
3. **Evitar consultas sin **``** en **``** o **`` para prevenir modificaciones masivas no deseadas.
4. **Usar tablas temporales** para operaciones complejas y evitar bloqueos.
5. **Probar en entornos de desarrollo** antes de ejecutar en producción.

---
