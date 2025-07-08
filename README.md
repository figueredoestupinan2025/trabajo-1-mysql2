5. Subconsultas
1. Consultar el producto más caro en cada categoría.
 SELECT nombre_contacto, tipo_id, precio
     FROM productos
     WHERE (tipo_id, precio) IN (
         SELECT tipo_id, MAX(precio)
         FROM productos
         GROUP BY tipo_id
     );
+-----------------+---------+---------+
| nombre_contacto | tipo_id | precio  |
+-----------------+---------+---------+
| servilleta      |       1 | 1500.00 |
+-----------------+---------+---------+
2. Encontrar el cliente con mayor total en pedidos.
SELECT nombre FROM clientes
     WHERE id = (
         SELECT cliente_id
         FROM pedidos
         GROUP BY cliente_id
         ORDER BY SUM(total) DESC
         LIMIT 1
    );
+--------+
| nombre |
+--------+
| camilo |
+--------+
3. Listar empleados que ganan más que el salario promedio.
SELECT id, nombre, salario
     FROM datosempleados
     WHERE salario > (
         SELECT AVG(salario) FROM datosempleados
     );
+----+---------+------------+
| id | nombre  | salario    |
+----+---------+------------+
|  1 | gerente | 1230000.00 |
+----+---------+------------+
4. Consultar productos que han sido pedidos más de 5 veces.
SELECT producto_id, COUNT(*) AS veces
     FROM detallepedido
     GROUP BY producto_id
     HAVING veces > 5;
5. Listar pedidos cuyo total es mayor al promedio de todos los pedidos.
SELECT id, total
     FROM pedidos
     WHERE total > (
         SELECT AVG(total) FROM pedidos
     );
+----+---------+
| id | total   |
+----+---------+
|  1 | 2000.00 |
+----+---------+
6. Seleccionar los 3 proveedores con más productos.
SELECT proveedor_id, COUNT(*) AS total
    FROM productos
    GROUP BY proveedor_id
    ORDER BY total DESC
    LIMIT 3;
+--------------+-------+
| proveedor_id | total |
+--------------+-------+
|            1 |    12 |
+--------------+-------+
7. Consultar productos con precio superior al promedio en su tipo.
 SELECT id, nombre_contacto, precio
    FROM productos p
    WHERE precio > (
        SELECT AVG(p2.precio)
        FROM productos p2
        WHERE p2.tipo_id = p.tipo_id
    );
+----+-----------------+---------+
| id | nombre_contacto | precio  |
+----+-----------------+---------+
|  8 | servilleta      | 1500.00 |
+----+-----------------+---------+
8. Mostrar clientes que han realizado más pedidos que la media.
SELECT id, nombre
    FROM clientes
    WHERE id IN (
        SELECT cliente_id
        FROM pedidos
        GROUP BY cliente_id
        HAVING COUNT(*) > (
            SELECT AVG(total_pedidos)
            FROM (
                SELECT cliente_id, COUNT(*) AS total_pedidos
                FROM pedidos
                GROUP BY cliente_id
            ) AS sub
        )
    );
+----+--------+
| id | nombre |
+----+--------+
|  1 | camilo |
+----+--------+
9. Encontrar productos cuyo precio es mayor que el promedio de todos los productos.
 SELECT id, nombre_contacto, precio
    FROM productos
    WHERE precio > (
        SELECT AVG(precio) FROM productos
    );
+----+-----------------+---------+
| id | nombre_contacto | precio  |
+----+-----------------+---------+
|  1 | jabon           |  500.00 |
|  8 | servilleta      | 1500.00 |
+----+-----------------+---------+
10. Mostrar empleados cuyo salario es menor al promedio del departamento.
SELECT e.id, e.nombre, e.salario
    FROM datosempleados e
    WHERE salario < (
        SELECT AVG(salario)
        FROM datosempleados
        WHERE puesto_id = e.puesto_id
    );
+----+-------------+---------+
| id | nombre      | salario |
+----+-------------+---------+
|  2 | ana lopez   | 2200.00 |
|  3 | andres ruiz | 2500.00 |
+----+-------------+---------+
6. Procedimientos Almacenados
1. Crear un procedimiento para actualizar el precio de todos los productos de un proveedor.
DELIMITER //
CREATE PROCEDURE actualizar_precio_proveedor(
    IN prov_id INT,
    IN nuevo_precio DECIMAL(10,2)
)
BEGIN
    UPDATE productos SET precio = nuevo_precio WHERE proveedor_id = prov_id;
END //
DELIMITER ;

-- USO:
CALL actualizar_precio_proveedor(1, 999.99);

2. Un procedimiento que devuelva la dirección de un cliente por ID.
DELIMITER //
CREATE PROCEDURE obtener_direccion_cliente(
    IN cliente_id INT
)
BEGIN
    SELECT u.direccion, u.ciudad, u.estado, u.pais
    FROM clientes c
    JOIN ubicaciones u ON c.ubicacion_id = u.id
    WHERE c.id = cliente_id;
END //
DELIMITER ;

-- USO:
CALL obtener_direccion_cliente(1);

3. Crear un procedimiento que registre un pedido nuevo y sus detalles.
DELIMITER //
CREATE PROCEDURE registrar_pedido(
    IN cliente_id INT,
    IN empleado_id INT,
    IN fecha DATE,
    IN producto_id INT,
    IN cantidad INT,
    IN precio_unitario DECIMAL(10,2)
)
BEGIN
    DECLARE nuevo_id INT;

    INSERT INTO pedidos (cliente_id, empleado_id, fecha, total)
    VALUES (cliente_id, empleado_id, fecha, cantidad * precio_unitario);

    SET nuevo_id = LAST_INSERT_ID();

    INSERT INTO detallepedido (pedido_id, producto_id, cantidad, precio)
    VALUES (nuevo_id, producto_id, cantidad, precio_unitario);
END //
DELIMITER ;

-- USO:
CALL registrar_pedido(1, 2, '2025-07-07', 3, 2, 250.00);

4. Un procedimiento para calcular el total de ventas de un cliente.
DELIMITER //
CREATE PROCEDURE total_ventas_cliente(
    IN cliente_id INT
)
BEGIN
    SELECT c.nombre, SUM(p.total) AS total_ventas
    FROM clientes c
    JOIN pedidos p ON c.id = p.cliente_id
    WHERE c.id = cliente_id
    GROUP BY c.id;
END //
DELIMITER ;

-- USO:
CALL total_ventas_cliente(1);

5. Crear un procedimiento para obtener los empleados por puesto.
DELIMITER //
CREATE PROCEDURE empleados_por_puesto(
    IN nombre_puesto VARCHAR(50)
)
BEGIN
    SELECT e.nombre
    FROM datosempleados e
    JOIN puestos p ON e.puesto_id = p.id
    WHERE p.nombre = nombre_puesto;
END //
DELIMITER ;

-- USO:
CALL empleados_por_puesto('administrador');

6. Un procedimiento que actualice el salario de empleados por puesto.
DELIMITER //
CREATE PROCEDURE actualizar_salario_por_puesto(
    IN nombre_puesto VARCHAR(50),
    IN nuevo_salario DECIMAL(10,2)
)
BEGIN
    UPDATE datosempleados e
    JOIN puestos p ON e.puesto_id = p.id
    SET e.salario = nuevo_salario
    WHERE p.nombre = nombre_puesto;
END //
DELIMITER ;

-- USO:
CALL actualizar_salario_por_puesto('administrador', 3000.00);

7. Crear un procedimiento que liste los pedidos entre dos fechas.
DELIMITER //
CREATE PROCEDURE pedidos_entre_fechas(
    IN fecha_inicio DATE,
    IN fecha_fin DATE
)
BEGIN
    SELECT * FROM pedidos
    WHERE fecha BETWEEN fecha_inicio AND fecha_fin;
END //
DELIMITER ;

-- USO:
CALL pedidos_entre_fechas('2025-06-01', '2025-07-01');

8. Un procedimiento para aplicar un descuento a productos de una categoría.
DELIMITER //
CREATE PROCEDURE aplicar_descuento_categoria(
    IN tipo_categoria VARCHAR(100),
    IN porcentaje_descuento DECIMAL(5,2)
)
BEGIN
    UPDATE productos p
    JOIN tipoproductos t ON p.tipo_id = t.id
    SET p.precio = p.precio * (1 - porcentaje_descuento / 100)
    WHERE t.tipo_nombre = tipo_categoria;
END //
DELIMITER ;

-- USO:
CALL aplicar_descuento_categoria('aseo', 10);

9. Crear un procedimiento que liste todos los proveedores de un tipo de producto.
DELIMITER //
CREATE PROCEDURE proveedores_por_tipo(
    IN tipo_nombre VARCHAR(100)
)
BEGIN
    SELECT DISTINCT prov.nombre
    FROM proveedores prov
    JOIN productos pr ON pr.proveedor_id = prov.id
    JOIN tipoproductos t ON pr.tipo_id = t.id
    WHERE t.tipo_nombre = tipo_nombre;
END //
DELIMITER ;

-- USO:
CALL proveedores_por_tipo('aseo');

10. Un procedimiento que devuelva el pedido de mayor valor.
DELIMITER //
CREATE PROCEDURE pedido_mayor_valor()
BEGIN
    SELECT p.id, p.total, c.nombre AS cliente
    FROM pedidos p
    JOIN clientes c ON p.cliente_id = c.id
    ORDER BY p.total DESC
    LIMIT 1;
END //
DELIMITER ;

-- USO:
CALL pedido_mayor_valor();