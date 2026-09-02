# API Specification: Menu Management

Este documento especifica la API para la gestión del menú digital, relacionada con la Historia de Usuario #1: "Como cliente, quiero ver un menú digital interactivo para seleccionar mis productos fácilmente".

## Base URL
```
https://api.restosmart.com/api/v1
```

## Autenticación
Todas las rutas requieren autenticación mediante JWT en el header:
```
Authorization: Bearer <token>
```

> Nota: Para la visualización del menú por parte de clientes no autenticados, se puede usar una ruta pública especial o tokens temporales.

## Endpoints

### 1. Obtener Categorías del Menú
Obtiene todas las categorías de productos disponibles.

**GET** `/menu/categories`

#### Respuesta
```json
[
  {
    "id": 1,
    "name": "Entradas",
    "description": "Platos para iniciar la comida",
    "image_url": "https://example.com/images/categories/entradas.jpg",
    "display_order": 1,
    "is_active": true
  },
  {
    "id": 2,
    "name": "Platos Fuertes",
    "description": "Platos principales",
    "image_url": "https://example.com/images/categories/platos_fuertes.jpg",
    "display_order": 2,
    "is_active": true
  },
  {
    "id": 3,
    "name": "Postres",
    "description": "Dulces y postres",
    "image_url": "https://example.com/images/categories/postres.jpg",
    "display_order": 3,
    "is_active": true
  }
]
```

#### Códigos de Respuesta
- `200 OK`: Categorías obtenidas exitosamente
- `500 Internal Server Error`: Error del servidor

### 2. Obtener Productos por Categoría
Obtiene todos los productos de una categoría específica.

**GET** `/menu/categories/{categoryId}/products`

#### Parámetros de Ruta
- `categoryId` (integer): ID de la categoría

#### Parámetros de Query (Opcionales)
- `available_only` (boolean): Si es true, solo devuelve productos disponibles (por defecto: true)
- `search` (string): Término de búsqueda en nombre y descripción

#### Respuesta
```json
[
  {
    "id": 101,
    "name": "Ensalada César",
    "description": "Lechuga romana, pollo a la parrilla, crutones, queso parmesano y aderezo César",
    "price": 12.50,
    "image_url": "https://example.com/images/products/ensalada_cesar.jpg",
    "category_id": 1,
    "is_available": true,
    "preparation_time": 10, // minutos
    "calories": 420,
    "ingredients": ["lechuga romana", "pollo", "crutones", "queso parmesano", "aderezo César"],
    "allergens": ["lacteo", "gluten"],
    "display_order": 1,
    "modifiers": [
      {
        "id": 1,
        "name": "Adicional de Pollo",
        "price": 3.00,
        "type": "addon",
        "max_selections": 1
      },
      {
        "id": 2,
        "name": "Sin Crutones",
        "price": 0.00,
        "type": "remove",
        "max_selections": 1
      }
    ]
  }
]
```

#### Códigos de Respuesta
- `200 OK`: Productos obtenidos exitosamente
- `404 Not Found`: Categoría no encontrada
- `500 Internal Server Error`: Error del servidor

### 3. Obtener Detalles de un Producto
Obtiene la información detallada de un producto específico.

**GET** `/menu/products/{productId}`

#### Parámetros de Ruta
- `productId` (integer): ID del producto

#### Respuesta
(Mismo formato que un elemento en el array del endpoint anterior)

#### Códigos de Respuesta
- `200 OK`: Producto obtenido exitosamente
- `404 Not Found`: Producto no encontrado
- `500 Internal Server Error`: Error del servidor

### 4. Crear un Pedido (Relacionado con Historia de Usuario #2)
Crea un nuevo pedido basado en las selecciones del menú.

**POST** `/orders`

#### Cuerpo de la Solicitud
```json
{
  "customer_info": {
    "name": "Juan Pérez",
    "phone": "+56912345678",
    "email": "juan.perez@email.com",
    "table_number": 5, // Para consumo en local, null para para llevar
    "takeaway": false
  },
  "items": [
    {
      "product_id": 101,
      "quantity": 2,
      "modifiers": [
        {
          "modifier_id": 1,
          "quantity": 1
        }
      ],
      "special_instructions": "Extra aderezo al lado"
    },
    {
      "product_id": 205,
      "quantity": 1,
      "modifiers": [],
      "special_instructions": "Poco salado"
    }
  ],
  "tip_amount": 3.00, // Opcional
  "discount_code": "DESCUENTO10" // Opcional
}
```

#### Respuesta
```json
{
  "id": 1001,
  "order_number": "ORD-20260902-001",
  "status": "pending",
  "total_amount": 28.00,
  "subtotal": 25.00,
  "tax_amount": 2.50,
  "tip_amount": 3.00,
  "discount_amount": 0.00,
  "estimated_preparation_time": 25, // minutos
  "estimated_ready_time": "2026-09-02T20:30:00Z",
  "created_at": "2026-09-02T20:05:00Z",
  "items": [
    {
      "product_id": 101,
      "product_name": "Ensalada César",
      "quantity": 2,
      "unit_price": 12.50,
      "total_price": 25.00,
      "modifiers": [
        {
          "name": "Adicional de Pollo",
          "quantity": 1,
          "price": 3.00
        }
      ],
      "special_instructions": "Extra aderezo al lado"
    }
  ]
}
```

#### Códigos de Respuesta
- `201 Created`: Pedido creado exitosamente
- `400 Bad Request`: Datos de entrada inválidos
- `402 Payment Required`: Fallo en el procesamiento de pago
- `500 Internal Server Error`: Error del servidor

## Modelos de Datos

### Categoría
```json
{
  "id": "integer",
  "name": "string",
  "description": "string",
  "image_url": "string (url)",
  "display_order": "integer",
  "is_active": "boolean"
}
```

### Producto
```json
{
  "id": "integer",
  "name": "string",
  "description": "string",
  "price": "decimal",
  "image_url": "string (url)",
  "category_id": "integer",
  "is_available": "boolean",
  "preparation_time": "integer (minutos)",
  "calories": "integer",
  "ingredients": "string[]",
  "allergens": "string[]",
  "display_order": "integer",
  "modifiers": "Modifier[]"
}
```

### Modificador
```json
{
  "id": "integer",
  "name": "string",
  "price": "decimal",
  "type": "string (addon/remove/substitute)",
  "max_selections": "integer"
}
```

### Pedido
```json
{
  "id": "integer",
  "order_number": "string (formato: ORD-YYYYMMDD-XXX)",
  "status": "string (pending/preparing/ready/delivered/cancelled)",
  "customer_info": "CustomerInfo",
  "items": "OrderItem[]",
  "subtotal": "decimal",
  "tax_amount": "decimal",
  "tip_amount": "decimal",
  "discount_amount": "decimal",
  "total_amount": "decimal",
  "estimated_preparation_time": "integer (minutos)",
  "estimated_ready_time": "string (ISO datetime)",
  "created_at": "string (ISO datetime)",
  "updated_at": "string (ISO datetime)"
}
```

### CustomerInfo
```json
{
  "name": "string",
  "phone": "string",
  "email": "string",
  "table_number": "integer (nullable)",
  "takeaway": "boolean"
}
```

### OrderItem
```json
{
  "product_id": "integer",
  "product_name": "string",
  "quantity": "integer",
  "unit_price": "decimal",
  "total_price": "decimal",
  "modifiers": "ModifierDetail[]",
  "special_instructions": "string (nullable)"
}
```

### ModifierDetail
```json
{
  "name": "string",
  "quantity": "integer",
  "price": "decimal"
}
```

## Códigos de Estado HTTP
- `200 OK`: Solicitud exitosa
- `201 Created`: Recurso creado exitosamente
- `400 Bad Request`: Solicitud mal formada o datos inválidos
- `401 Unauthorized`: Falta de autenticación o token inválido
- `403 Forbidden`: No autorizado para acceder al recurso
- `404 Not Found`: Recurso no encontrado
- `409 Conflicto`: Conflicto con el estado actual del recurso
- `422 Unprocessable Entity`: Error de validación semántica
- `429 Too Many Requests`: Límite de tasa excedido
- `500 Internal Server Error`: Error interno del servidor
- `503 Service Unavailable`: Servicio temporalmente no disponible

## Manejo de Errores
Todos los errores devuelven un cuerpo JSON con el siguiente formato:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Descripción legible del error",
    "details": {} // Información adicional opcional
  }
}
```

Ejemplo:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "El precio debe ser un número positivo",
    "details": {
      "field": "price",
      "value": "-5.00",
      "rule": "min_value"
    }
  }
}
```

## Rate Limiting
- Límite estándar: 100 solicitudes por minuto por IP
- Endpoints autenticados: 500 solicitudes por minuto por usuario
- Los encabezados de respuesta incluyen:
  - `X-RateLimit-Limit`: Límite máximo
  - `X-RateLimit-Remaining`: Solicitudes restantes
  - `X-RateLimit-Reset`: Tiempo de reinicio en segundos desde epoch

## Versionamiento
La API usa versionamiento en la URL (`/api/v1/`). Los cambios que rompen compatibilidad incrementarán la versión mayor.