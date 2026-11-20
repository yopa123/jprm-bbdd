# Guía del Dataset de Prueba: Tienda Retail (1M+ Registros)

Este paquete de prueba simula la operación integral de una cadena de retail omnicanal, incluyendo tiendas físicas, catálogo de productos, clientes geolocalizados, más de un millón de órdenes de compra y un grafo topológico de interacciones.

---

## 1. Estructura y Volumen de Datos

| Componente | Tipo | Registros | Descripción y Casos de Uso |
| :--- | :--- | :--- | :--- |
| **`stores`** | Tabla Columnar | 15 | Sucursales físicas con nombre, ciudad y coordenadas `(lat, lon)` para pruebas con **GeoSQL**. |
| **`products`** | Tabla Columnar | 1.000 | Catálogo de productos (SKU, nombre, categoría, marca, precio, stock). |
| **`customers`** | Tabla Columnar | 50.000 | Clientes con ciudad, coordenadas geográficas, `lifetime_value` y categorización `is_vip`. |
| **`orders`** | Tabla Columnar | **1.000.000** | Transacciones de compra (monto total, ítems, canal, medio de pago, estado). |
| **`graph.redb`** | Grafo Topológico | **500.000** | Enlaces de red en almacenamiento ACID B-Tree (`COMPRO_PRODUCTO`, `ASISTIO_TIENDA`, `RECOMENDO_A`). |

---

## 2. Puesta en Marcha con el Dataset

Para iniciar el motor cargando directamente este dataset:

```bash
# En Linux
./jprm-bbdd-server --data-dir ./sample_data --port 8123

# En Windows
.\jprm-bbdd-server.exe --data-dir .\sample_data --port 8123
```

El servidor detectará y registrará automáticamente todas las tablas Parquet y el grafo ACID.

---

## 3. Consultas SQL de Ejemplo para Probar el Motor

### A. Analítica Columnar de Alto Rendimiento (1 Millón de Órdenes)
Agrupación y cálculo financiero sobre el millón de transacciones en memoria:

```sql
SELECT 
    channel,
    payment_method,
    COUNT(*) AS total_ventas,
    ROUND(SUM(total_amount), 2) AS ingreso_total,
    ROUND(AVG(total_amount), 2) AS ticket_promedio
FROM orders
WHERE status = 'completed'
GROUP BY channel, payment_method
ORDER BY ingreso_total DESC;
```

---

### B. Geofencing y Filtro Espacial (GeoSQL)
Encontrar todos los clientes que viven a menos de **5 km** (5.000 metros) de la tienda **Costanera Center** (`lat: -33.4172`, `lon: -70.6067`):

```sql
SELECT 
    c.customer_id, 
    c.name, 
    c.city,
    ROUND(ST_DistanceSphere(c.lat, c.lon, -33.4172, -70.6067), 0) AS distancia_metros
FROM customers c
WHERE ST_DWithin(c.lat, c.lon, -33.4172, -70.6067, 5000.0)
ORDER BY distancia_metros ASC
LIMIT 10;
```

---

### C. Consulta Híbrida: Grafo Topológico + Tablas (`GRAPH_PATH`)
Descubrir qué clientes compraron el producto **ID 7** a través del grafo topológico (Relación `101: COMPRO_PRODUCTO`) y unirlo con la tabla de clientes para segmentar a los compradores VIP:

```sql
SELECT 
    c.customer_id, 
    c.name, 
    c.email, 
    c.city, 
    c.lifetime_value
FROM GRAPH_PATH(INCOMING, 7, 101) g
JOIN customers c ON g.node_id = c.customer_id
WHERE c.is_vip = TRUE
ORDER BY c.lifetime_value DESC
LIMIT 15;
```

---

### D. Red de Recomendaciones Virales (Clientes que Recomendaron a Clientes)
Buscar la red de recomendaciones del cliente **ID 1052** (Relación `301: RECOMENDO_A`):

```sql
SELECT 
    c.customer_id, 
    c.name, 
    c.lifetime_value
FROM GRAPH_PATH(OUTGOING, 1052, 301) g
JOIN customers c ON g.node_id = c.customer_id;
```
