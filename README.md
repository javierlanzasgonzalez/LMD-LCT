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
