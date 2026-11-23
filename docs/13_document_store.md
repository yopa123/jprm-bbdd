# jprm-BBDD — Document Store (MongoDB Style)

jprm-BBDD incorpora un motor de documentos transaccional schemaless de ultra-baja latencia respaldado en un árbol B-Tree ACID (`redb`) con serialización binaria compacta (MessagePack).

Permite almacenar objetos JSON libres de esquema, consultarlos con sintaxis familiar estilo MongoDB a través de HTTP REST, y además **cruzarlos nativamente con Grafos topológicos y Tablas analíticas** usando SQL estándar.

---

## 🚀 1. Endpoints HTTP (Estilo MongoDB / Document Store)

El servidor expone rutas dedicadas para trabajar con colecciones y documentos JSON:

### 1.1. Insertar un Documento
- **Método:** `POST /collections/{coleccion}/insert`
- **Headers:** `Content-Type: application/json`
- **Body:** Cualquier objeto JSON válido. Si incluye `_id` o `id`, se usa como identificador único; si no, el motor genera un UUID v4 automáticamente.

```bash
curl -X POST http://localhost:8123/collections/agentes_ia/insert \
  -H "Content-Type: application/json" \
  -d '{
    "_id": "agente_01",
    "modelo": "llama-3-local",
    "temperatura": 0.7,
    "contexto": { "rol": "asistente", "idioma": "es-cl" },
    "tags": ["ia", "local", "chile"]
  }'
```
**Respuesta:**
```json
{
  "status": "ok",
  "collection": "agentes_ia",
  "inserted_id": "agente_01"
}
```

---

### 1.2. Obtener un Documento por ID
- **Método:** `GET /collections/{coleccion}/{id}`

```bash
curl -s http://localhost:8123/collections/agentes_ia/agente_01
```
**Respuesta:**
```json
{
  "id": "agente_01",
  "data": {
    "_id": "agente_01",
    "modelo": "llama-3-local",
    "temperatura": 0.7,
    "contexto": { "rol": "asistente", "idioma": "es-cl" },
    "tags": ["ia", "local", "chile"]
  },
  "created_at": 1795218007548,
  "updated_at": 1795218007548
}
```

---

### 1.3. Buscar Documentos con Filtro (`find`)
- **Método:** `POST /collections/{coleccion}/find`
- **Body:** Filtro con las propiedades que deben coincidir exactamente:

```bash
curl -X POST http://localhost:8123/collections/agentes_ia/find \
  -H "Content-Type: application/json" \
  -d '{ "modelo": "llama-3-local" }'
```

---

### 1.4. Listar Colecciones y Eliminar Documentos
- **Listar colecciones:** `GET /collections`
- **Eliminar por ID:** `DELETE /collections/{coleccion}/{id}`

---

## ⚡ 2. Funciones JSON en Consultas SQL

Puedes extraer propiedades de documentos JSON directamente en cualquier consulta SQL:

| Función SQL | Descripción | Ejemplo |
| :--- | :--- | :--- |
| `json_extract(json_str, path)` | Extrae un valor o sub-objeto JSON como texto | `json_extract(data, '$.contexto.rol')` |
| `json_extract_float(json_str, path)` | Extrae un número flotante para cálculos matemáticos | `json_extract_float(data, '$.temperatura')` |
| `json_array_contains(json_str, path, elem)` | Verifica si un array contiene un elemento determinado | `json_array_contains(data, 'tags', 'chile')` |

```sql
SELECT 
    _id,
    json_extract(data, '$.modelo') AS modelo,
    json_extract_float(data, '$.temperatura') AS temp
FROM COLLECTION('agentes_ia')
WHERE json_array_contains(data, 'tags', 'chile');
```

---

## 🔺 3. El Triángulo Híbrido: Documentos ↔ Grafos ↔ Tablas

El verdadero poder surge cuando cruzas los tres motores en una sola sentencia SQL sin mover datos de servidor:

```sql
-- Caso de uso: Recomendar productos a un cliente basándose en:
-- 1. Metadata y preferencias del cliente (Documento JSON)
-- 2. Conexiones en la red social de amigos (Grafo Topológico)
-- 3. Historial de compras masivas (Tabla Columnar Parquet)

SELECT 
    d._id AS cliente_id,
    json_extract(d.data, '$.nombre') AS nombre,
    json_extract(d.data, '$.preferencias.plan') AS plan,
    SUM(v.monto_compra) AS total_gastado
FROM clientes_ia d
JOIN recomendados_grafo g ON CAST(d._id AS BIGINT) = g.node_id
JOIN ventas v ON d._id = v.customer_id
GROUP BY d._id, d.data;
```
