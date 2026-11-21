# jprm-BBDD

<p align="center">
  <img src="https://img.shields.io/badge/Versi%C3%B3n-1.0.1--alpha-6366f1?style=for-the-badge" alt="Versión 1.0.1-alpha" />
  <img src="https://img.shields.io/badge/Plataforma-Windows%20x64%20%7C%20Linux%20x64-10b981?style=for-the-badge" alt="Plataforma x64" />
  <img src="https://img.shields.io/badge/Estado-Experimental-8b5cf6?style=for-the-badge" alt="Experimental" />
</p>

> **Motor de base de datos híbrido**.  
> Diseñado y desarrollado por Juan Pablo R.M. como un proyecto experimental en mis tiempos libres, pensado para acompañar y potenciar el ecosistema del lenguaje de programación JPRM.  
> Unifica en un solo servicio almacenamiento columnar para consultas analíticas (OLAP), navegación de grafos/redes topológicas y colecciones de documentos JSON con búsqueda vectorial usando SQL estándar.

---

## ⚡ Sobre este proyecto

Este motor nació de la curiosidad y de las ganas de explorar nuevos conceptos de diseño de datos durante mis ratos libres. El objetivo fue armar un binario único, liviano y fácil de desplegar que permita trabajar con tres modelos en una sola herramienta:

- **Consultas Analíticas en Memoria:** Procesar grandes volúmenes de datos con almacenamiento columnar eficiente.
- **Grafos y Redes en SQL:** Navegar relaciones y conexiones entre entidades mediante un simple `JOIN`.
- **Colecciones de Documentos y Vectores:** Documentos JSON schemaless con índices secundarios y similitud vectorial para GraphRAG.
- **Cálculo Geoespacial Integrado:** Resolver distancias y áreas geográficas en metros de forma directa.
- **Acceso Directo por HTTP y CLI:** Una interfaz sencilla para enviar consultas SQL desde cualquier lenguaje (Python, Node.js, curl, el lenguaje JPRM o su CLI interactivo).

> ⚠️ **Estado del Proyecto y Descargo de Responsabilidad:**  
> Este motor es un proyecto personal experimental (versión alpha) en desarrollo activo durante mis tiempos libres. Se entrega "tal cual" (*AS IS*), con propósitos estrictamente educativos, de investigación y experimentación.  
> **No me hago responsable por pérdidas de datos, fallos, interrupciones o cualquier daño directo o indirecto que pudiera ocurrir al utilizar este software.** Utilícelo bajo su propia responsabilidad y mantenga siempre respaldos externos.

---

## 📐 El Triángulo de Motores: ¿Cómo se conectan?

En lugar de montar tres sistemas separados (por ejemplo ClickHouse para análisis tabular, Neo4j para relaciones de red y MongoDB para documentos JSON), **jprm-BBDD unifica los tres paradigmas en un único binario con memoria compartida Zero-Copy:**

```text
                           ▲ [TABULAR ANALÍTICO]
                          ╱ ╲ Apache Arrow (SIMD) + DataFusion
                         ╱   ╲ Almacenamiento Columnar Parquet
                        ╱  ★  ╲ Agregaciones de alta velocidad
                       ╱       ╲
                      ╱  TRIÁNGULO ╲
                     ╱   UNIFICADO  ╲
                    ╱                ╲
                   ╱   Zero-Copy      ╲
                  ╱    Plan Engine     ╲
                 ╱                      ╲
 [GRAFOS TOPOLÓGICOS] ◀──────────────────▶ [DOCUMENTAL SCHEMALESS]
  redb B-Tree Key-Value                     redb Multi-Collection + JSON
  Vecinos y saltos O(1)                     Índices B-Tree en O(log N)
  GRAPH_PATH() en SQL                       Vectores Cosine/SIMD & GraphRAG
```

### ¿Cómo interactúan los tres vértices?

1. **Tabular Analítico (Parquet + Apache Arrow):**
   - Ejecuta agrupaciones, métricas numéricas y filtros masivos con vectorización SIMD multihilo en nanosegundos.
2. **Grafos Topológicos (B-Tree en `redb`):**
   - Resuelve relaciones complejas, jerarquías y caminos de red (`GRAPH_PATH`) en microsegundos sin hacer costosos joins relacionales de múltiples tablas.
3. **Documental y Vectorial (MessagePack en `redb`):**
   - Guarda entidades dinámicas con campos arbitrarios, indexa propiedades JSON en $O(\log N)$ y calcula distancias vectoriales (`cosine_similarity`) para búsquedas semánticas.

#### El "Triangle Join" (Cruce Híbrido en una sola consulta)
El planificador del motor permite cruzar los 3 mundos en una sola sentencia SQL estándar:

```sql
SELECT 
    d._id,
    json_extract(d.data, '$.nombre') AS cliente,
    t.monto,
    g.weight AS afinidad_red
FROM COLLECTION('clientes_ia') d
JOIN GRAPH_PATH(OUTGOING, 101, 1) g ON CAST(json_extract(d.data, '$.nodo_id') AS BIGINT) = g.target_id
JOIN sales t ON t.customer_id = g.target_id
WHERE cosine_similarity(d.embedding, '[0.12, 0.95, 0.44]') > 0.85;
```

---

## 🚀 Capacidades del Motor

- **Almacenamiento Columnar Eficiente:** Formato columnar comprimido y procesamiento vectorial en memoria para consultas rápidas.
- **Colecciones de Documentos (Schemaless):** Almacenamiento JSON flexible de ultra-baja latencia con API REST estilo MongoDB (`/collections`), inserciones atómicas por lotes (`insert_batch`), índices secundarios y funciones de extracción JSON en SQL (`json_extract`, `json_array_contains`).
- **Búsqueda Vectorial y GraphRAG:** Funciones nativas (`cosine_similarity`, `euclidean_distance`, `dot_product`) para IA local combinada con topología de grafos.
- **Navegación de Redes con `GRAPH_PATH`:** Motor de grafos transaccional integrado para resolver caminos y conexiones en microsegundos.
- **Autoguardado y Compacting (Vacuum):** Mantenimiento continuo en segundo plano y comando `VACUUM` para recuperar espacio libre de disco.
- **Replicación Streaming WAL (Master-Replica):** Registro secuencial de mutaciones con LSN y endpoint `/replication/stream` para réplicas de lectura.
- **CLI Interactivo Propio (`jprm-bbdd-cli`):** Terminal interactiva tipo `psql` / `mongosh` con historial, tablas ASCII y sintaxis documental.
- **GeoSQL Integrado:** Funciones geodésicas directas (`ST_DWithin`, `ST_DistanceSphere`) y compatibilidad con geometrías estándar (`POINT`).
- **Tablas Federadas (Zero-ETL):** Comando `CREATE EXTERNAL TABLE` para consultar datos remotos directamente sin tener que copiarlos localmente.
- **Sincronización Incremental:** Comando `SYNC TABLE` para mantener tablas al día trayendo únicamente los cambios recientes.
- **Vistas Materializadas con Refresco en Segundo Plano:** `CREATE MATERIALIZED VIEW ... REFRESH EVERY N SECONDS` para precalcular reportes de forma continua.
- **Respaldos Atómicos en Caliente (`BACKUP DATABASE`):** Generación de respaldos consistentes con verificación criptográfica SHA-256 sin pausar ni bloquear el servicio.
- **Control de Acceso (RBAC):** Autenticación mediante tokens y permisos configurables por usuario.

---

## 🖥️ Plataformas Soportadas

Distribuimos binarios precompilados de 64 bits listos para usar sin depender de librerías externas del sistema:

| Sistema Operativo | Arquitectura | Archivo Binario | Requisitos |
| :--- | :--- | :--- | :--- |
| **Windows** | x86_64 / x64 | `jprm-bbdd-server.exe` / `jprm-bbdd-cli.exe` | Windows 10, 11 o Windows Server 2016+ |
| **Linux** | x86_64 | `jprm-bbdd-server` / `jprm-bbdd-cli` | Distribuciones con glibc 2.31+ (Ubuntu 20.04+, Debian 11+, Rocky/RHEL 8+) |

---

## 🏁 Inicio Rápido

### 1. Iniciar el Servidor

Arráncalo indicando el puerto y la carpeta donde guardará los datos:

**En Windows:**
```powershell
.\jprm-bbdd-server.exe --port 8123 --data-dir .\jprm_bbdd_data
```

**En Linux:**
```bash
chmod +x ./jprm-bbdd-server
./jprm-bbdd-server --port 8123 --data-dir ./jprm_bbdd_data
```

Para verificar que esté arriba:
```bash
curl http://localhost:8123/ping
# Retorna: ok
```

---

### 2. Usar la Terminal Interactiva (`jprm-bbdd-cli`)

Puedes conectarte con el CLI oficial interactivo:

```powershell
.\jprm-bbdd-cli.exe --url http://localhost:8123
```

Dentro de la terminal puedes ejecutar consultas SQL, sintaxis estilo Mongo y metacomandos:
```javascript
// Crear o insertar documentos
db.clientes.insert({ "nombre": "Juan", "rol": "arquitecto", "ciudad": "Santiago" })

// Búsqueda con índice secundario
db.clientes.find({ "rol": "arquitecto" })

// Mantenimiento y estado
\status
\wal
\vacuum
\tables
```

---

### 3. Probar Consultas SQL Vía HTTP

Puedes enviarle consultas directamente por HTTP POST con `curl` o PowerShell:

```bash
# Crear una tabla
curl -s -X POST http://localhost:8123/ -d "CREATE TABLE sensores (id INT, ubicacion VARCHAR, temperatura DOUBLE, lat DOUBLE, lon DOUBLE)"

# Insertar datos
curl -s -X POST http://localhost:8123/ -d "INSERT INTO sensores VALUES (1, 'Sucursal Costanera', 21.4, -33.4172, -70.6067)"
curl -s -X POST http://localhost:8123/ -d "INSERT INTO sensores VALUES (2, 'Sucursal Parque Arauco', 19.8, -33.4022, -70.5786)"

# Consultar
curl -s -X POST http://localhost:8123/ -d "SELECT * FROM sensores WHERE temperatura > 20.0"
```

---

### 4. Ejemplo Geoespacial y de Grafos

**Filtro por radio geográfico (sensores a menos de 3.000 metros):**
```sql
SELECT id, ubicacion, temperatura
FROM sensores
WHERE ST_DWithin(lat, lon, -33.4172, -70.6067, 3000.0);
```

**Cruce con red de grafos (`GRAPH_PATH`):**
```sql
SELECT s.ubicacion, g.node_id AS cliente_relacionado
FROM GRAPH_PATH(OUTGOING, 1, 101) g
JOIN sensores s ON g.node_id = s.id;
```

---

## 📚 Base de Conocimiento y Documentación

La documentación técnica detallada está organizada en la carpeta [`docs/`](docs/):

1. **[Configuración del Motor (`jprm-bbdd.toml`)](docs/01_configuration.md):** Parámetros de red, buffers de memoria, timeouts y rutas.
2. **[Referencia SQL y Tipos de Datos](docs/02_sql_reference.md):** Tipos admitidos, DDL, DML y análisis con `EXPLAIN`.
3. **[Funciones Geoespaciales (GeoSQL)](docs/03_geospatial.md):** Distancias esféricas, geofencing y coordenadas WKT.
4. **[Topología de Grafos (`GRAPH_PATH`)](docs/04_graph_queries.md):** Relaciones entrantes, salientes y joins relacionales.
5. **[Ingesta Masiva y Zero-ETL](docs/05_ingestion_and_zero_etl.md):** Carga y federación desde Postgres, MySQL, SQL Server, SQLite y archivos.
6. **[Sincronización Incremental CDC](docs/06_incremental_sync.md):** Réplicas delta automáticas por columna de control.
7. **[Vistas Materializadas](docs/07_materialized_views.md):** Precalculo de consultas y programación de refresco automático.
8. **[Seguridad y Roles (RBAC)](docs/08_security_rbac.md):** Creación de usuarios, tokens Bearer y permisos.
9. **[Respaldos en Caliente (Backup)](docs/09_backup_and_recovery.md):** Snapshots atómicos, archivo manifest y restauración.
10. **[Métricas y Monitoreo](docs/10_monitoring_metrics.md):** Healthcheck `/ping`, telemetría `/status` y Prometheus `/metrics`.
11. **[SDK Oficial de Python y Conectores](docs/11_integrations_and_sdk.md):** Ejemplos prácticos para Python, Node.js, PHP y curl.
12. **[Dataset de Prueba Retail (1M+ Registros)](docs/12_retail_sample_dataset.md):** Guía del paquete de prueba con 1.000.000 de órdenes y grafo cargado.
13. **[Motor de Documentos (Tipo MongoDB)](docs/13_document_store.md):** Persistencia JSON schemaless, endpoints REST `/collections` y consultas híbridas del triángulo.

---

## ⚖️ Términos de Uso

### Versión Alpha (`v1.0.1-alpha`)
Esta versión está pensada para pruebas de arquitectura, desarrollo y benchmarking en mis tiempos libres. Aunque el motor guarda los datos de forma transaccional y consistente en disco, en cualquier entorno mantén siempre tus políticas de respaldo activas.

### Descargo Legal y Licencia
El autor **no se hace responsable por pérdidas de información, daños directos o indirectos, lucro cesante o cualquier perjuicio derivado del uso, prueba o instalación de este software.** Se distribuye sin garantías de ninguna especie.

Los ejecutables de **jprm-BBDD** se distribuyen compilados bajo evaluación e integración personal. Todas las dependencias de terceros de código abierto se detallan con sus licencias en la documentación.
