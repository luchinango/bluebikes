# Diccionario de Datos – Bluebikes Boston
**Versión:** 1.0  
**Fecha:** 2025-08-22  
**Autoría:** Equipo de proyecto Bluebikes

Este documento describe los **datasets y campos** principales del repositorio, su **tipo**, **reglas de calidad**, **dominios**, **unidades** y **derivación** cuando aplica. Está pensado para incluirse en `DATA_DICTIONARY.md` del repositorio de GitHub.

---

## Estructura de carpetas sugerida
```
data/
  raw/            # archivos fuente originales (sin modificar)
  interim/        # integraciones temporales / staging
  curated/        # tabla de viajes limpia y unificada (parquet)
  features/       # tabla de viajes con features derivadas
  aggregates/     # agregados por estación x hora
notebooks/
  01_Proyecto_final_Blue_Bikes_Limpieza-EDA.ipynb
  02_Proyecto_final_Blue_Bikes_Ing_Caracteristicas.ipynb
models/           # artefactos de entrenamiento (si aplica)
docs/             # reportes e imágenes
```

---

## Dataset: **curated/trips.parquet** (tabla limpia y minable)
| Campo | Tipo | Dominio / Unidades | Nulos | Descripción | Derivación / Regla | Ejemplo |
|---|---|---|:---:|---|---|---|
| `ride_id` | string | — | No | Identificador único de viaje (si faltante en esquemas legados, se genera hash). | Directo o generado. | `C9A1B23F` |
| `rideable_type` | enum | `classic_bike` \| `electric_bike` \| `docked_bike` | No* | Tipo de bicicleta. | Si nulo ⇒ **`docked_bike`**. | `classic_bike` |
| `started_at` | timestamp | hora local provista por fuente | No | Timestamp de inicio del viaje (sin zona horaria). | Cast a `timestamp`. | `2023-07-15 08:23:10` |
| `ended_at` | timestamp | hora local provista por fuente | No | Timestamp de fin del viaje. | Cast a `timestamp`. | `2023-07-15 08:37:12` |
| `duration_sec` | int | segundos | No | Duración del viaje. | `ended_at - started_at`; **60 ≤ duración ≤ 86.400**. | `842` |
| `start_station_name` | string | — | No | Estación de inicio (o etiqueta). | Si nulo ⇒ **`Dockless start`**. | `South Station - 700 Atlantic Ave` |
| `end_station_name` | string | — | No | Estación de fin (o etiqueta). | Si nulo ⇒ **`Dockless end`**. | `Copley Square - Dartmouth St` |
| `start_lat` | double | grados | No | Latitud del punto de inicio. | Filtrado por BBOX+buffer del área operativa. | `42.3523` |
| `start_lng` | double | grados | No | Longitud del punto de inicio. | Filtrado por BBOX+buffer del área operativa. | `-71.0552` |
| `end_lat` | double | grados | No | Latitud del punto de fin. | Filtrado por BBOX+buffer. | `42.3491` |
| `end_lng` | double | grados | No | Longitud del punto de fin. | Filtrado por BBOX+buffer. | `-71.0776` |
| `member_casual` | enum | `member` \| `casual` | No | Tipo de usuario. | Directo. | `member` |
| `schema_version` | string | `legacy` \| `intermediate` \| `recent` (o similar) | No | Versión de esquema detectada al integrar. | Set por pipeline. | `recent` |
| `periodo` | string | `YYYY-MM` | No | Año-mes para particionado/consulta. | Derivado de `started_at_local`. | `2023-07` |

**Reglas de calidad (curated):**  
- Eliminar viajes con `duration_sec < 60` o `> 86.400`.  
- Eliminar registros sin coordenadas finales y con coordenadas 0/0.  
- Filtrar viajes con puntos fuera del área operativa (BBOX + buffer Boston/Salem).  

---

## Dataset: **features/trips_with_features.parquet** (viajes con features)
| Campo | Tipo | Dominio / Unidades | Nulos | Descripción / Derivación | Ejemplo |
|---|---|---|:---:|---|---|
| `started_at_local` | timestamp tz | America/New_York | No | `started_at` con zona horaria aplicada. | `2023-07-15 08:23:10-04:00` |
| `ts_hour` | timestamp tz | hora truncada | No | `date_trunc('hour', started_at_local)` | `2023-07-15 08:00:00-04:00` |
| `trip_date` | date | — | No | `date(started_at_local)` | `2023-07-15` |
| `trip_year` | int | 2015…2025 | No | Año del viaje. | `2023` |
| `trip_month` | int | 1–12 | No | Mes del viaje. | `7` |
| `trip_day_of_week` | int | 0–6 (0=Lun…6=Dom) | No | Día de semana (convención ISO modificada). | `5` |
| `trip_hour` | int | 0–23 | No | Hora del día. | `8` |
| `is_weekend` | bool | `true/false` | No | `trip_day_of_week in (5,6)` | `true` |
| `hour_sin` | float | [-1,1] | No | `sin(2π*trip_hour/24)` | `0.5` |
| `hour_cos` | float | [-1,1] | No | `cos(2π*trip_hour/24)` | `0.866` |
| `day_of_week_sin` | float | [-1,1] | No | `sin(2π*trip_day_of_week/7)` | `-0.781` |
| `day_of_week_cos` | float | [-1,1] | No | `cos(2π*trip_day_of_week/7)` | `0.624` |
| `haversine_distance_km` | float | km | No | Distancia geodésica Haversine entre inicio y fin. | `2.3` |
| `avg_speed_kmh` | float | km/h | No | `(haversine_distance_km) / (duration_sec/3600)` | `9.8` |
| `is_round_trip` | bool | `true/false` | No | `start_station_name = end_station_name` (o distancia < 50 m en dockless). | `false` |
| `is_popular_start` | bool | `true/false` | No | Estación en el percentil ≥ 90 de partidas. | `true` |
| `is_holiday` | bool | `true/false` | No | Feriados Massachusetts (`holidays.US(subdiv="MA")`). | `false` |
| `season` | enum | `winter/spring/summer/fall` | No | Mapeo por mes: DJF=invierno, MAM=primavera, JJA=verano, SON=otoño. | `summer` |

---

## Dataset: **aggregates/hourly_station.parquet** (agregado estación × hora)
| Campo | Tipo | Descripción | Ejemplo |
|---|---|---|---|
| `station_name` | string | Identificador de estación (o etiqueta dockless). | `South Station - 700 Atlantic Ave` |
| `ts_hour` | timestamp tz | Ventana horaria local. | `2023-07-15 08:00:00-04:00` |
| `rides` | int | Viajes totales en la hora. | `37` |
| `unique_trips` | int | Viajes únicos (habitualmente = `rides`). | `37` |
| `avg_dur_sec` | float | Duración media (s). | `780.5` |
| `median_dur_sec` | float | Duración mediana (s). | `720` |
| `avg_speed_kmh` | float | Velocidad media (km/h). | `10.3` |
| `km_total` | float | Kilómetros totales sumados en la hora. | `85.7` |
| `round_trips` | int | Viajes ida/vuelta en misma estación. | `2` |
| `rides_member` | int | Viajes de usuarios miembros. | `29` |
| `rides_casual` | int | Viajes de usuarios casuales. | `8` |
| `rides_classic_bike` | int | Viajes con classic. | `25` |
| `rides_electric_bike` | int | Viajes con e‑bike. | `12` |
| `rides_docked_bike` | int | Viajes con docked. | `0` |
| `trip_month` | int | Mes (1–12). | `7` |
| `trip_day_of_week` | int | 0–6. | `5` |
| `trip_hour` | int | 0–23. | `8` |
| `is_weekend` | bool | Fin de semana. | `true` |
| `is_holiday` | bool | Feriado en MA. | `false` |
| `hour_sin` | float | Codificación cíclica. | `0.5` |
| `hour_cos` | float | Codificación cíclica. | `0.866` |
| `day_of_week_sin` | float | Codificación cíclica. | `-0.781` |
| `day_of_week_cos` | float | Codificación cíclica. | `0.624` |
| `season` | enum | Estación del año. | `summer` |

**Definición de agregación:**  
- Granularidad: **estación × hora local (`ts_hour`)**.  
- Métrica objetivo para modelado: `rides`.  
- Lags y ventanas (para dataset de entrenamiento): `rides_lag_1h`, `rides_lag_24h`, `rides_lag_168h`, `rides_ma_3h`, `rides_ma_6h`, `rides_ma_12h`, `rides_ma_24h`, `rides_ma_168h` (no persistidos por defecto).

---

## Reglas y validaciones de calidad (Data Quality)
- **Duración:** `60 ≤ duration_sec ≤ 86.400`.  
- **Coordenadas:** no nulas; exclusión de puntos 0/0; dentro de BBOX+buffer del área operativa.  
- **Estaciones nulas:** etiquetar como `Dockless start/end` para preservar viajes.  
- **Tipos:** conversión consistente y formatos estables para particionado por `periodo`.  
- **Dominio `rideable_type` y `member_casual`:** restringidos a los enumerados.

Ejemplos de *checks* (dbt‑style):
```sql
-- Dominio rideable_type
select rideable_type from curated.trips
where rideable_type not in ('classic_bike','electric_bike','docked_bike');

-- Rango de duración
select * from curated.trips
where duration_sec < 60 or duration_sec > 86400;
```

---

## Linaje (Lineage) resumido
- `curated/trips.parquet` ⇐ integración multi‑esquema + limpieza (Spark).  
- `features/trips_with_features.parquet` ⇐ `curated/trips` + enriquecimiento temporal/geométrico.  
- `aggregates/hourly_station.parquet` ⇐ `features/trips_with_features` agregado por `station_name, ts_hour` y pivote de `rideable_type`/`member_casual`.

---

## Consideraciones de privacidad y campos descartados
- Campos legados `birth_year`, `gender`, `postal_code` presentan alta ausencia y **no** se incluyen en `curated/` por bajo valor analítico y para minimizar riesgos de re‑identificación.  
- No se manejan datos personales directos (PII).

---

## Control de versiones del diccionario
- **1.0 (actual):** Definiciones para capas `curated`, `features` y `aggregates`; reglas de calidad y dominios.
