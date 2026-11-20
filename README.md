# jprm-BBDD

<p align="center">
  <img src="https://img.shields.io/badge/Versi%C3%B3n-1.0.0--alpha-6366f1?style=for-the-badge" alt="Versión 1.0.0-alpha" />
  <img src="https://img.shields.io/badge/Plataforma-Windows%20x64%20%7C%20Linux%20x64-10b981?style=for-the-badge" alt="Plataforma x64" />
  <img src="https://img.shields.io/badge/Estado-Experimental-8b5cf6?style=for-the-badge" alt="Experimental" />
</p>

> **Motor de base de datos híbrido**.  
> Diseñado y desarrollado por Juan Pablo R.M. como un proyecto experimental en mis tiempos libres, pensado para acompañar y potenciar el ecosistema del lenguaje de programación JPRM.  
> Unifica en un solo servicio almacenamiento columnar para consultas analíticas (OLAP), navegación de grafos/redes topológicas y cálculos geoespaciales usando SQL estándar.

---

## ⚡ Sobre este proyecto

Este motor nació de la curiosidad y de las ganas de explorar nuevos conceptos de diseño de datos durante mis ratos libres. El objetivo fue armar un binario único, liviano y fácil de desplegar que permita trabajar con tres modelos en una sola herramienta:

- **Consultas Analíticas en Memoria:** Procesar grandes volúmenes de datos con almacenamiento columnar eficiente.
- **Grafos y Redes en SQL:** Navegar relaciones y conexiones entre entidades mediante un simple `JOIN`.
- **Cálculo Geoespacial Integrado:** Resolver distancias y áreas geográficas en metros de forma directa.
- **Acceso Directo por HTTP:** Una interfaz sencilla para enviar consultas SQL desde cualquier lenguaje (Python, Node.js, curl o el lenguaje JPRM).

> ⚠️ **Estado del Proyecto:** Este motor se encuentra actualmente en fase experimental (versión alpha). Está pensado para fines de investigación, desarrollo y experimentación personal.

---

## 🚀 Capacidades del Motor

- **Almacenamiento Columnar Eficiente:** Formato columnar comprimido y procesamiento vectorial en memoria para consultas rápidas.
- **Colecciones de Documentos (Schemaless):** Almacenamiento JSON flexible de ultra-baja latencia con API REST estilo MongoDB (`/collections`) y funciones de extracción JSON en SQL (`json_extract`, `json_array_contains`).
- **Navegación de Redes con `GRAPH_PATH`:** Motor de grafos transaccional integrado para resolver caminos y conexiones en microsegundos.
- **GeoSQL Integrado:** Funciones geodésicas directas (`ST_DWithin`, `ST_DistanceSphere`) y compatibilidad con geometrías estándar (`POINT`).
- **Tablas Federadas (Zero-ETL):** Comando `CREATE EXTERNAL TABLE` para consultar datos remotos directamente sin tener que copiarlos localmente.
- **Sincronización Incremental:** Comando `SYNC TABLE` para mantener tablas al día trayendo únicamente los cambios recientes.
- **Vistas Materializadas con Refresco en Segundo Plano:** `CREATE MATERIALIZED VIEW ... REFRESH EVERY N SECONDS` para precalcular reportes de forma continua.
- **Respaldos Atómicos en Caliente (`BACKUP DATABASE`):** Generación de respaldos consistentes sin pausar ni bloquear el servicio.
- **Control de Acceso (RBAC):** Autenticación mediante tokens y permisos configurables por usuario.

---

## 🖥️ Plataformas Soportadas

Distribuimos binarios precompilados de 64 bits listos para usar sin depender de librerías externas del sistema:

| Sistema Operativo | Arquitectura | Archivo Binario | Requisitos |
| :--- | :--- | :--- | :--- |
| **Windows** | x86_64 / x64 | `jprm-bbdd-server.exe` | Windows 10, 11 o Windows Server 2016+ |
| **Linux** | x86_64 | `jprm-bbdd-server` | Distribuciones con glibc 2.31+ (Ubuntu 20.04+, Debian 11+, Rocky/RHEL 8+) |

Los paquetes comprimidos se descargan desde la sección de [Releases de GitHub](https://github.com/yopa123/jprm-bbdd/releases).

---

## 🏁 Inicio Rápido

### 1. Iniciar el Servidor

Descarga el release para tu sistema operativo y arráncalo indicando el puerto y la carpeta donde guardará los datos:

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

### 2. Probar Consultas SQL

Puedes mandarle consultas directamente por HTTP POST con `curl` o PowerShell:

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

### 3. Ejemplo Geoespacial y de Grafos

**Filtro por radio geográfico (clientes a menos de 3.000 metros):**
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

Si usas asistentes de IA como Cursor, Claude Code o Copilot, revisa [`llms.txt`](llms.txt), que contiene la guía de sintaxis y arquitectura condensada.

---

## ⚖️ Términos de Uso

### Versión Alpha (`v1.0.0-alpha`)
Esta versión está pensada para pruebas de arquitectura, desarrollo y benchmarking. Aunque el motor guarda los datos de forma transaccional y consistente en disco, en ambientes de producción mantén siempre tus políticas de respaldo activas.

### Licencia de Distribución Binaria
Los ejecutables de **jprm-BBDD** se distribuyen compilados bajo licencia privativa para evaluación e integración de sistemas. No está permitida la ingeniería inversa ni la redistribución comercial no autorizada de los binarios. Todas las dependencias de terceros de código abierto se detallan con sus licencias en [NOTICE-THIRD-PARTY.txt](NOTICE-THIRD-PARTY.txt).
