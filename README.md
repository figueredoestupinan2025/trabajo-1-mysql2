📘 Proyecto SQL: Consultas Avanzadas y Procedimientos Almacenados
Este proyecto incluye subconsultas avanzadas y procedimientos almacenados diseñados para manipular y consultar eficientemente una base de datos con múltiples relaciones. Las tablas principales consideradas son:

productos

clientes

pedidos

detallepedido

datosempleados

puestos

tipoproductos

proveedores

ubicaciones

🧠 5. Subconsultas
1. Producto más caro en cada categoría
sql
Copiar
Editar
SELECT nombre_contacto, tipo_id, precio
FROM productos
WHERE (tipo_id, precio) IN (
    SELECT tipo_id, MAX(precio)
    FROM productos
    GROUP BY tipo_id
);
2. Cliente con mayor total en pedidos
sql
Copiar
Editar
SELECT nombre FROM clientes
WHERE id = (
    SELECT cliente_id
    FROM pedidos
    GROUP BY cliente_id
    ORDER BY SUM(total) DESC
    LIMIT 1
);
3. Empleados con salario superior al promedio
sql
Copiar
Editar
SELECT id, nombre, salario
FROM datosempleados
WHERE salario > (
    SELECT AVG(salario) FROM datosempleados
);
4. Productos pedidos más de 5 veces
sql
Copiar
Editar
SELECT producto_id, COUNT(*) AS veces
FROM detallepedido
GROUP BY producto_id
HAVING veces > 5;
5. Pedidos cuyo total es mayor al promedio general
sql
Copiar
Editar
SELECT id, total
FROM pedidos
WHERE total > (
    SELECT AVG(total) FROM pedidos
);
6. Los 3 proveedores con más productos
sql
Copiar
Editar
SELECT proveedor_id, COUNT(*) AS total
FROM productos
GROUP BY proveedor_id
ORDER BY total DESC
LIMIT 3;
7. Productos con precio mayor al promedio de su tipo
sql
Copiar
Editar
SELECT id, nombre_contacto, precio
FROM productos p
WHERE precio > (
    SELECT AVG(p2.precio)
    FROM productos p2
    WHERE p2.tipo_id = p.tipo_id
);
8. Clientes con más pedidos que la media
sql
Copiar
Editar
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
9. Productos cuyo precio es superior al promedio general
sql
Copiar
Editar
SELECT id, nombre_contacto, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio) FROM productos
);
10. Empleados con salario menor al promedio de su departamento
sql
Copiar
Editar
SELECT e.id, e.nombre, e.salario
FROM datosempleados e
WHERE salario < (
    SELECT AVG(salario)
    FROM datosempleados
    WHERE puesto_id = e.puesto_id
);
⚙️ 6. Procedimientos Almacenados
1. Actualizar el precio de productos de un proveedor
sql
Copiar
Editar
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
2. Obtener dirección de cliente por ID
sql
Copiar
Editar
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
3. Registrar un pedido con detalles
sql
Copiar
Editar
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
4. Calcular el total de ventas de un cliente
sql
Copiar
Editar
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
5. Obtener empleados por puesto
sql
Copiar
Editar
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
6. Actualizar salario por puesto
sql
Copiar
Editar
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
7. Listar pedidos entre fechas
sql
Copiar
Editar
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
8. Aplicar descuento a productos de una categoría
sql
Copiar
Editar
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
9. Listar proveedores por tipo de producto
sql
Copiar
Editar
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
10. Pedido de mayor valor
sql
Copiar
Editar
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
