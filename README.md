### 1. ¿Cuántas filas devuelve cada consulta y por qué son distintas? Explicá con ejemplos concretos de los datos qué filas se eliminaron con UNION.

* **Consulta 1 (`UNION` - Catálogo de productos):** Devuelve **11 filas**.
* **Consulta 2 (`UNION ALL` - Auditoría completa de stock):** Devuelve **14 filas**.

#### **¿Por qué son distintas?**
* **`UNION ALL`** combina las 7 filas de la Sucursal Norte con las 7 filas de la Sucursal Sur sin realizar ningún filtro, devolviendo el total exacto de registros cargados (7 + 7 = 14 filas).
* **`UNION`** compara los resultados de ambas tablas y elimina aquellas filas cuyos valores seleccionados sean idénticos en todas las columnas.

#### **Ejemplos concretos de los datos:**
Al seleccionar `id_producto`, `nombre_producto` y `categoria` para armar el catálogo comercial:
1. **Monitor 4K 27" (`id_producto = 103`, `Computación`):** Existe en ambas sucursales con los mismos atributos de producto. `UNION` detecta que la fila es idéntica y conserva una sola.
2. **Teclado Mecánico (`id_producto = 104`, `Accesorios`):** Aparece exactamente igual en ambas tablas. `UNION` elimina el duplicado.
3. **SSD Externo 1TB (`id_producto = 106`, `Almacenamiento`):** También coincide plenamente entre Norte y Sur. `UNION` fusiona ambas filas en una sola.

> **Nota técnica sobre la Webcam (`107` vs `111`):** 
> La *Webcam HD 1080p* tiene `id_producto = 107` en el Norte e `id_producto = 111` en el Sur. Como sus ID no coinciden, `UNION` no la considera duplicada y la conserva como dos filas distintas en el catálogo.

---

### 2. ¿Por qué UNION ALL es más eficiente que UNION? ¿Qué operación adicional realiza UNION internamente que consume más recursos?

`UNION ALL` es más rápido y eficiente porque simplemente **concatena** los conjuntos de datos a medida que los lee de la base de datos.

Por el contrario, **`UNION`** realiza un trabajo computacional extra:
1. Combina los registros de ambas fuentes.
2. **Operación adicional (Ordenamiento / De-duplicación):** Aplica internamente un proceso de ordenamiento de datos (*Sort*) o una tabla hash (*Hash Match / Distinct*) para comparar fila por fila en memoria/disco y eliminar las repetidas.

Este proceso de búsqueda y eliminación de duplicados consume más procesador (CPU), memoria RAM y operaciones de I/O en la base de datos.

---

### 3. ¿En qué casos de negocio usarías cada uno? Dá al menos dos ejemplos reales distintos a los del ejercicio.

#### **Casos para usar `UNION ALL`:**
1. **Consolidación de transacciones históricas (Logs / Ventas):** Unir la tabla de `ventas_2023` y `ventas_2024` para calcular la facturación total. Nos interesa mantener cada ticket individual.
2. **Métricas de tráfico web:** Unir registros de *clicks* o visitas desde la app móvil y el sitio web para contar el total absoluto de interacciones.

#### **Casos para usar `UNION`:**
1. **Base unificada de clientes (Email Marketing):** Unir la lista de clientes de e-commerce con la de tiendas físicas para enviar un newsletter sin mandar correos duplicados.
2. **Directorio telefónico de proveedores:** Combinar las agendas de proveedores de distintas filiales para obtener un listado maestro único de contactos activos.

---

### 4. ¿Qué pasa si las columnas de ambas consultas no coinciden en número o tipo? ¿Qué error genera SQL?

SQL exige estrictamente que ambas consultas unidas tengan la misma estructura:

1. **Si no coinciden en el número de columnas:**
   SQL Server devuelve el error:
   `Msg 205, Level 16, State 1: All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.`
2. **Si los tipos de datos son incompatibles:**
   Si intentás unir una columna `INT` con un `VARCHAR` no convertible, SQL devuelve el error:
   `Msg 245, Level 16, State 1: Conversion failed when converting the varchar value 'Texto' to data type int.`
