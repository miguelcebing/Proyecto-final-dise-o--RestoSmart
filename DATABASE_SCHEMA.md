# Database Schema: Menu and Order Management

Este documento describe el esquema de base de datos propuesto para la gestión de menús y pedidos, relacionado con las Historias de Usuario #1 y #2.

## Esquema General

La base de datos sigue un diseño relacional normalizado con las siguientes entidades principales:
- Categorías de productos
- Productos
- Modificadores (opciones de personalización)
- Pedidos
- Ítems de pedido
- Información del cliente
- Pagos

## Tablas

### 1. categories (Categorías)
Almacena las diferentes categorías de productos del menú.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| name | VARCHAR(100) | NOT NULL | Nombre de la categoría |
| description | TEXT | NULLABLE | Descripción detallada |
| image_url | VARCHAR(255) | NULLABLE | URL de la imagen de la categoría |
| display_order | INTEGER | NOT NULL, DEFAULT 0 | Orden de visualización en el menú |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | Indica si la categoría está activa |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_categories_active (is_active) WHERE is_active = TRUE
- INDEX idx_categories_display_order (display_order)

### 2. products (Productos)
Almacena los productos disponibles en el menú.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| category_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES categories(id) | ID de la categoría a la que pertenece |
| name | VARCHAR(200) | NOT NULL | Nombre del producto |
| description | TEXT | NULLABLE | Descripción detallada |
| price | DECIMAL(10,2) | NOT NULL, CHECK (price >= 0) | Precio base del producto |
| image_url | VARCHAR(255) | NULLABLE | URL de la imagen del producto |
| is_available | BOOLEAN | NOT NULL, DEFAULT TRUE | Indica si el producto está disponible para ordenar |
| preparation_time | INTEGER | NOT NULL, CHECK (preparation_time >= 0) | Tiempo estimado de preparación en minutos |
| calories | INTEGER | NULLABLE, CHECK (calories >= 0) | Cantidad aproximada de calorías |
| display_order | INTEGER | NOT NULL, DEFAULT 0 | Orden de visualización dentro de la categoría |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_products_category (category_id)
- INDEX idx_products_available (is_available) WHERE is_available = TRUE
- INDEX idx_products_display_order (display_order)
- FULLTEXT INDEX ft_products_search (name, description) (para búsquedas de texto)

### 3. modifiers (Modificadores / Opciones de Personalización)
Almacena las opciones disponibles para personalizar los productos (adiciones, eliminaciones, sustituciones).

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| name | VARCHAR(100) | NOT NULL | Nombre del modificador |
| price | DECIMAL(10,2) | NOT NULL | Precio adicional/costo del modificador (puede ser negativo para descuentos) |
| type | ENUM('addon', 'remove', 'substitute') | NOT NULL | Tipo de modificador |
| max_selections | INTEGER | NOT NULL, CHECK (max_selections >= 0) | Máximo número de veces que se puede seleccionar este modificador (0 = ilimitado) |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | Indica si el modificador está activo |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_modifiers_active (is_active) WHERE is_active = TRUE
- INDEX idx_modifiers_type (type)

### 4. product_modifiers (Relación Producto-Modificador)
Define qué modificadores están disponibles para cada producto.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| product_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES products(id) ON DELETE CASCADE | ID del producto |
| modifier_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES modifiers(id) ON DELETE CASCADE | ID del modificador |
| is_default | BOOLEAN | NOT NULL, DEFAULT FALSE | Indica si este modificador viene seleccionado por defecto |
| display_order | INTEGER | NOT NULL, DEFAULT 0 | Orden de visualización en la lista de modificadores |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |

**Índices:**
- PRIMARY KEY (product_id, modifier_id)
- INDEX idx_product_modifiers_product (product_id)
- INDEX idx_product_modifiers_modifier (modifier_id)
- INDEX idx_product_modifiers_default (is_default) WHERE is_default = TRUE

### 5. customers (Clientes)
Almacena información de los clientes (puede ser usado tanto para clientes registrados como para información temporal de pedidos).

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| name | VARCHAR(150) | NOT NULL | Nombre completo del cliente |
| phone | VARCHAR(20) | NULLABLE | Número de teléfono |
| email | VARCHAR(255) | NULLABLE, UNIQUE | Correo electrónico |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_customers_email (email) WHERE email IS NOT NULL
- INDEX idx_customers_phone (phone) WHERE phone IS NOT NULL

### 6. orders (Pedidos)
Almacena la información principal de los pedidos.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| order_number | VARCHAR(20) | NOT NULL, UNIQUE | Número de pedido formateado (ej: ORD-20260902-001) |
| customer_id | INTEGER | NULLABLE, FOREIGN KEY REFERENCES customers(id) | ID del cliente (NULL para clientes no registrados/guest) |
| table_number | INTEGER | NULLABLE | Número de mesa (para consumo en local) |
| takeaway | BOOLEAN | NOT NULL, DEFAULT FALSE | Indica si es para llevar o consumo en local |
| status | ENUM('pending', 'confirmed', 'preparing', 'ready', 'delivered', 'cancelled') | NOT NULL, DEFAULT 'pending' | Estado actual del pedido |
| subtotal | DECIMAL(10,2) | NOT NULL, CHECK (subtotal >= 0) | Subtotal antes de impuestos, propina y descuentos |
| tax_amount | DECIMAL(10,2) | NOT NULL, CHECK (tax_amount >= 0) | Monto de impuestos |
| tip_amount | DECIMAL(10,2) | NOT NULL, CHECK (tip_amount >= 0) | Monto de propina |
| discount_amount | DECIMAL(10,2) | NOT NULL, CHECK (discount_amount >= 0) | Monto de descuentos aplicados |
| total_amount | DECIMAL(10,2) | NOT NULL, CHECK (total_amount >= 0) | Monto total a pagar |
| estimated_preparation_time | INTEGER | NULLABLE, CHECK (estimated_preparation_time >= 0) | Tiempo estimado de preparación total en minutos |
| estimated_ready_time | DATETIME | NULLABLE | Fecha y hora estimada de completion |
| actual_ready_time | DATETIME | NULLABLE | Fecha y hora real de completion |
| special_instructions | TEXT | NULLABLE | Instrucciones especiales generales para el pedido |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- UNIQUE KEY uk_order_number (order_number)
- INDEX idx_orders_customer (customer_id)
- INDEX idx_orders_status (status)
- INDEX idx_orders_created_at (created_at)
- INDEX idx_orders_table_number (table_number) WHERE table_number IS NOT NULL
- INDEX idx_orders_takeaway (takeaway)
- INDEX idx_orders_status_time (status, created_at) para consultas de pedidos activos recientes

### 7. order_items (Ítems de Pedido)
Almacena los productos específicos incluidos en cada pedido.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| order_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES orders(id) ON DELETE CASCADE | ID del pedido al que pertenece |
| product_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES products(id) | ID del producto |
| quantity | INTEGER | NOT NULL, CHECK (quantity >= 1) | Cantidad del producto solicitada |
| unit_price | DECIMAL(10,2) | NOT NULL, CHECK (unit_price >= 0) | Precio unitario al momento del pedido |
| total_price | DECIMAL(10,2) | NOT NULL, CHECK (total_price >= 0) | Precio total para este ítem (quantity × unit_price) |
| special_instructions | TEXT | NULLABLE | Instrucciones específicas para este ítem |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_order_items_order (order_id)
- INDEX idx_order_items_product (product_id)
- INDEX idx_order_items_order_product (order_id, product_id)

### 8. order_item_modifiers (Modificadores aplicados a ítems de pedido)
Almacena qué modificadores se aplicaron a cada ítem en un pedido específico.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| order_item_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES order_items(id) ON DELETE CASCADE | ID del ítem de pedido |
| modifier_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES modifiers(id) | ID del modificador aplicado |
| quantity | INTEGER | NOT NULL, CHECK (quantity >= 1) | Cantidad de veces que se aplicó este modificador |
| price_per_unit | DECIMAL(10,2) | NOT NULL | Precio unitario del modificador al momento del pedido |
| total_price | DECIMAL(10,2) | NOT NULL, CHECK (total_price >= 0) | Precio total para este modificador en este ítem (quantity × price_per_unit) |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación |

**Índices:**
- PRIMARY KEY (id)
- INDEX idx_order_item_modifiers_item (order_item_id)
- INDEX idx_order_item_modifiers_modifier (modifier_id)
- INDEX idx_order_item_modifiers_item_modifier (order_item_id, modifier_id)

### 9. payments (Pagos)
Almacena la información de los pagos realizados por los pedidos.

| Columna | Tipo | Restricciones | Descripción |
|---------|------|---------------|-------------|
| id | INTEGER | PRIMARY KEY, AUTOINCREMENT | Identificador único |
| order_id | INTEGER | NOT NULL, FOREIGN KEY REFERENCES orders(id) ON DELETE CASCADE | ID del pedido asociado |
| payment_method | ENUM('credit_card', 'debit_card', 'cash', 'mobile_wallet', 'gift_card') | NOT NULL | Método de pago utilizado |
| transaction_id | VARCHAR(255) | NULLABLE, UNIQUE | ID de transacción del procesador de pagos |
| amount | DECIMAL(10,2) | NOT NULL, CHECK (amount >= 0) | Monto pagado |
| status | ENUM('pending', 'processing', 'completed', 'failed', 'refunded') | NOT NULL, DEFAULT 'pending' | Estado del pago |
| processed_at | DATETIME | NULLABLE | Fecha y hora en que fue procesado |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Fecha de creación del registro |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Fecha de última actualización |

**Índices:**
- PRIMARY KEY (id)
- UNIQUE KEY uk_payment_transaction (transaction_id) WHERE transaction_id IS NOT NULL
- INDEX idx_payments_order (order_id)
- INDEX idx_payments_status (status)
- INDEX idx_payments_method (payment_method)
- INDEX idx_payments_created_at (created_at)

## Relaciones entre Tablas

```
categories 1 ----< products
products 1 ----< order_items
products *> ----< product_modifiers *> ----< modifiers
orders 1 ----< order_items
orders 1 ----< payments
customers 1 ----< orders
order_items 1 ----< order_item_modifiers
modifiers 1 ----< order_item_modifiers
```

## Scripts de Inicialización de Datos de Ejemplo

```sql
-- Insertar categorías de ejemplo
INSERT INTO categories (name, description, display_order) VALUES
('Entradas', 'Platos para iniciar la comida', 1),
('Platos Fuertes', 'Platos principales', 2),
('Postres', 'Dulces y postres', 3),
('Bebidas', 'Refrescos, jugos y bebidas calientes', 4);

-- Insertar productos de ejemplo
INSERT INTO products (category_id, name, description, price, image_url, preparation_time, calories, display_order) VALUES
-- Entradas
(1, 'Ensalada César', 'Lechuga romana, pollo a la parrilla, crutones, queso parmesano y aderezo César', 12.50, 'https://example.com/images/products/ensalada_cesar.jpg', 10, 420, 1),
(1, 'Bruschetta Tomate y Albahaca', 'Pan tostado con tomate fresco, albahaca, ajo y aceite de oliva', 9.75, 'https://example.com/images/products/bruschetta.jpg', 8, 280, 2),
(1, 'Alitas BBQ', 'Alitas de pollo glaseadas con salsa BBQ', 14.00, 'https://example.com/images/products/alitas_bbq.jpg', 12, 650, 3),

-- Platos Fuertes
(2, 'Bife de Chorizo', 'Corte de 300g con papas fritas y ensalada mixta', 22.50, 'https://example.com/images/products/bife_chorizo.jpg', 18, 850, 1),
(2, 'Salmón a la Plancha', 'Filete de salmón con verduras al vapor y arroz integral', 24.00, 'https://example.com/images/products/salmon.jpg', 15, 520, 2),
(2, 'Ravioles de Ricotta y Espinaca', 'Ravioles caseros con salsa de tomate y albahaca', 18.75, 'https://example.com/images/products/ravioles.jpg', 12, 680, 3),

-- Postres
(3, 'Flan Casero', 'Flan tradicional con caramelo', 7.50, 'https://example.com/images/products/flan.jpg', 5, 280, 1),
(3, 'Tarta de Manzana', 'Porción generosa de tarta de manzana con helado de vainilla', 8.50, 'https://example.com/images/products/tarta_manzana.jpg', 10, 420, 2),
(3, 'Helado Artesanal', 'Copa de helado artesanal con toppings', 6.00, 'https://example.com/images/products/helado.jpg', 2, 350, 3),

-- Bebidas
(4, 'Agua Mineral', 'Botella de agua mineral sin gas', 2.50, 'https://example.com/images/products/agua.jpg', 2, 0, 1),
(4, 'Jugo de Naranja Natural', 'Jugo exprimido al momento', 4.00, 'https://example.com/images/products/jugo_naranja.jpg', 3, 120, 2),
(4, 'Café Especial', 'Café de origen único preparado por barista', 3.50, 'https://example.com/images/products/cafe.jpg', 4, 5, 3);

-- Insertar modificadores de ejemplo
INSERT INTO modifiers (name, price, type, max_selections) VALUES
-- Adiciones (Addons)
('Adicional de Pollo', 3.00, 'addon', 1),
('Doble Queso', 2.50, 'addon', 1),
('Huevo Frito', 1.50, 'addon', 1),
('Tocino Crujiente', 2.00, 'addon', 1),
('Guacamole', 2.25, 'addon', 1),
('Queso Feta', 1.75, 'addon', 1),

-- Eliminaciones (Removes)
('Sin Cebolla', 0.00, 'remove', 1),
('Sin Aglio', 0.00, 'remove', 1),
('Sin Picante', 0.00, 'remove', 1),
('Sin Lacteo', 0.00, 'remove', 1),
('Sin Gluten', 0.00, 'remove', 1),
('Sin Azúcar', 0.00, 'remove', 1),

-- Sustituciones (Substitutes)
('Pan Integral en lugar de Blanco', 0.50, 'substitute', 1),
('Arroz Integral en lugar de Blanco', 0.00, 'substitute', 1),
('Pasta Sin Gluten', 1.50, 'substitute', 1),
('Leche de Almendra', 0.75, 'substitute', 1);

-- Asociar modificadores a productos de ejemplo
-- Ensalada César puede tener adicional de pollo y sin crutones
INSERT INTO product_modifiers (product_id, modifier_id, is_default, display_order) VALUES
(101, 1, 0, 1), -- Ensalada César + Adicional de Pollo
(101, 7, 0, 2), -- Ensalada César + Sin Crutones (asumiendo que modifier_id 7 es "Sin Crutones")

-- Bife de Chorizo puede tener doble queso y huevo frito
INSERT INTO product_modifiers (product_id, modifier_id, is_default, display_order) VALUES
(201, 2, 0, 1), -- Bife de Chorizo + Doble Queso
(201, 3, 0, 2), -- Bife de Chorizo + Huevo Frito

-- Alitas BBQ pueden tener nivel de picante personalizable
INSERT INTO product_modifiers (product_id, modifier_id, is_default, display_order) VALUES
(203, 8, 0, 1), -- Alitas BBQ + Sin Picante
(203, 9, 0, 2), -- Alitas BBQ + Picante Extra (asumiendo que modifier_id 9 existe)
```

## Consideraciones de Rendimiento y Escalabilidad

### Índices Recomendados para Consultas Frecuentes

1. **Para la visualización del menú (Historias de Usuario #1):**
   ```sql
   -- Obtener categorías activas ordenadas
   SELECT * FROM categories WHERE is_active = TRUE ORDER BY display_order;
   
   -- Obtener productos de una categoría disponibles
   SELECT * FROM products 
   WHERE category_id = ? AND is_available = TRUE 
   ORDER BY display_order;
   
   -- Búsqueda de productos por nombre o descripción
   SELECT * FROM products 
   WHERE (name LIKE ? OR description LIKE ?) 
   AND is_available = TRUE 
   LIMIT 20;
   ```

2. **Para la creación de pedidos (Historias de Usuario #2):**
   ```sql
   -- Obtener información completa de un producto con sus modificadores
   SELECT p.*, 
          GROUP_CONCAT(DISTINCT m.id) AS modifier_ids,
          GROUP_CONCAT(DISTINCT m.name) AS modifier_names,
          GROUP_CONCAT(DISTINCT m.price) AS modifier_prices,
          GROUP_CONCAT(DISTINCT m.type) AS modifier_types
   FROM products p
   LEFT JOIN product_modifiers pm ON p.id = pm.product_id
   LEFT JOIN modifiers m ON pm.modifier_id = m.id AND m.is_active = TRUE
   WHERE p.id = ?
   GROUP BY p.id;
   
   -- Validar que los modificadores solicitados son válidos para el producto
   SELECT COUNT(*) AS valid_count
   FROM product_modifiers
   WHERE product_id = ? AND modifier_id IN (/* lista de IDs de modificadores solicitados */);
   ```

3. **Para el seguimiento de pedidos (Historias de Usuario #3 y #4):**
   ```sql
   -- Obtener pedidos por estado para cocina
   SELECT o.*, c.name AS customer_name, c.phone
   FROM orders o
   LEFT JOIN customers c ON o.customer_id = c.id
   WHERE o.status IN ('pending', 'confirmed', 'preparing')
   ORDER BY o.created_at ASC;
   
   -- Obtener pedidos listos para entrega
   SELECT o.*, c.name AS customer_name, c.phone, c.email
   FROM orders o
   LEFT JOIN customers c ON o.customer_id = c.id
   WHERE o.status = 'ready'
   ORDER BY o.estimated_ready_time ASC;
   ```

### Estrategias de Partitioning (para volúmenes altos)

1. **Partitioning por fecha en la tabla orders:**
   ```sql
   -- Particionar por rango de meses para mantener los últimos 12 meses activos
   PARTITION BY RANGE (YEAR(created_at)*100 + MONTH(created_at)) (
     PARTITION p202601 VALUES LESS THAN (202602),
     PARTITION p202602 VALUES LESS THAN (202603),
     -- ... continuar para cada mes
     PARTITION pmax VALUES LESS THAN MAXVALUE
   );
   ```

2. **Archivado de pedidos antiguos:**
   - Mover pedidos completados hace más de 2 años a una tabla de historial
   - Mantener solo los indices necesarios en la tabla activa

## Consideraciones de Seguridad

1. **Encriptación de Datos Sensibles:**
   - Información de pago completa nunca se almacena (solo los últimos 4 dígitos y ID de transacción)
   - Información de cliente se encripta en reposo si contiene datos sensibles adicionales

2. **Control de Acceso:**
   - Roles de base de datos diferenciados (lectura para frontend, lectura/escritura para backend)
   - Procedimientos almacenados para operaciones críticas cuando sea necesario

3. **Auditoría:**
   - Triggers para registrar cambios críticos en tablas de pedidos y pagos
   - Tabla de auditoría separada para cumplimiento regulatorio

## Scripts de Migración de Ejemplo

```sql
-- Agregar columna para tracking de origen del pedido
ALTER TABLE orders 
ADD COLUMN order_source ENUM('web', 'mobile_app', 'kiosk', 'staff') 
NOT NULL DEFAULT 'web' AFTER takeaway;

-- Agregar índice para consultas por origen
CREATE INDEX idx_orders_source ON orders(order_source);

-- Actualizar procedimientos existentes para incluir el nuevo campo
```

## Backup y Recuperación

1. **Backup Diario:**
   - Backup lógico completo cada 24 horas
   - Backups incrementales cada 4 horas para tablas de alto volumen (orders, order_items, payments)

2. **Estrategia de Punto de Recuperación:**
   - Binlog habilitado para recuperación punto-en-tiempo
   - Pruebas regulares de restauración

## Monitoreo y Métricas

### Métricas de Negocio a Rastrear
- Tiempo promedio desde pedido hasta preparación
- Tiempo promedio desde preparación hasta entrega
- Tasa de modificadores utilizados por pedido
- Productos más/menos personalizados
- Horarios pico de pedidos
- Tiempo promedio de decisión del menú

### Consultas para Dashboard de Cocina
```sql
-- Pedidos en espera de preparación
SELECT COUNT(*) AS pending_prep 
FROM orders 
WHERE status = 'pending';

-- Pedidos actualmente en preparación
SELECT COUNT(*) AS currently_preparing 
FROM orders 
WHERE status = 'preparing';

-- Tiempo promedio de preparación última hora
SELECT AVG(TIMESTAMPDIFF(MINUTE, created_at, updated_at)) AS avg_prep_time
FROM orders 
WHERE status = 'preparing' 
AND updated_at >= DATE_SUB(NOW(), INTERVAL 1 HOUR);

-- Pedidos que superan el tiempo estimado
SELECT COUNT(*) AS overdue_orders
FROM orders 
WHERE status = 'preparing' 
AND TIMESTAMPDIFF(MINUTE, created_at, NOW()) > 
    (SELECT AVG(preparation_time) FROM products WHERE id IN (
        SELECT product_id FROM order_items WHERE order_id = orders.id
    ));
```