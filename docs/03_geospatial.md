# GeoSQL & Geospatial Functions

jprm-BBDD includes native geodesic computing functions operating on contiguous 64-bit floating-point arrays. The algorithms use spherical trigonometry (Haversine formula with mean Earth radius $R = 6,371,000\text{ meters}$) to evaluate distance and geofences across millions of coordinates at hardware speeds.

---

## Supported Spatial Functions

### 1. `ST_DistanceSphere`
Computes the orthodromic (great-circle) distance between two pairs of latitude and longitude coordinates. Returns distance in **meters**.

```sql
-- Signature: ST_DistanceSphere(lat1, lon1, lat2, lon2) -> Float64
SELECT ST_DistanceSphere(-33.4489, -70.6693, -34.6037, -58.3816) AS distance_meters;
```

**Filtering by Distance in Queries:**
```sql
SELECT id, name, lat, lon,
       ST_DistanceSphere(lat, lon, -33.4489, -70.6693) AS distance_to_center
FROM stores
WHERE ST_DistanceSphere(lat, lon, -33.4489, -70.6693) <= 5000.0
ORDER BY distance_to_center ASC;
```

---

### 2. `ST_DWithin`
Evaluates whether two coordinate pairs are within a given radius in meters. Returns a `BOOLEAN` (`TRUE` / `FALSE`).

```sql
-- Signature: ST_DWithin(lat1, lon1, lat2, lon2, radius_meters) -> Boolean
SELECT id, vehicle_code, lat, lon
FROM fleet_vehicles
WHERE ST_DWithin(lat, lon, -33.4489, -70.6693, 2500.0);
```

> **Performance Tip**: When writing geofence filters inside a `WHERE` clause, prefer `ST_DWithin` over `ST_DistanceSphere(..) <= radius`. `ST_DWithin` avoids branch overhead and operates natively on columnar boolean masks.

---

### 3. `ST_Point`
Constructs a standard Well-Known Text (WKT) string representation from longitude and latitude coordinates:

```sql
-- Signature: ST_Point(lon, lat) -> VARCHAR ("POINT(lon lat)")
SELECT id, ST_Point(lon, lat) AS geom_wkt 
FROM delivery_points;
```

---

### 4. `ST_X` & `ST_Y`
Extracts longitude ($X$) or latitude ($Y$) from a WKT Point string:

```sql
-- Signature: ST_X(wkt_point) -> Float64
-- Signature: ST_Y(wkt_point) -> Float64
SELECT 
    ST_X(geom_wkt) AS longitude,
    ST_Y(geom_wkt) AS latitude
FROM assets;
```

---

## Coordinate Order Standard

- When passing individual numeric arguments to `ST_DistanceSphere` and `ST_DWithin`, the convention is **`(lat, lon)`**:
  `ST_DistanceSphere(lat1, lon1, lat2, lon2)`
- When constructing geometries with `ST_Point`, standard WKT Cartesian convention **`(X, Y) -> (lon, lat)`** is followed:
  `ST_Point(lon, lat)`
