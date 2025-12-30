# ADR 0003: Offline Dataset Schema and OSM Preprocessing Pipeline (V1)

## Status
Accepted

## Date
2025-04-02

## Context

V1 must operate fully offline on an Android tablet and provide low-latency speed limit lookups.
Raw OSM PBF is large and expensive to parse on-device. We therefore need:

- A repeatable, deterministic preprocessing pipeline run off-device (desktop/CI)
- A compact on-device dataset optimised for:
  - fast spatial candidate lookup
  - deterministic road selection (ADR 0002)
  - minimal storage footprint
- Clear data versioning and provenance so builds are auditable and testable

Northern Ireland is the initial target region; expansion to Great Britain and the Republic of Ireland may follow.

---

## Decision

### 1. Preprocess OSM PBF off-device into a SQLite database

The canonical on-device dataset shall be a single SQLite database file (per region) containing:

- road segment geometry (polyline)
- explicit speed limit (normalised)
- derived metadata to support deterministic matching
- spatial index (R*Tree) for candidate search

The Android app shall only perform reads from this database.

---

### 2. Use a “segmented roads” representation rather than full OSM ways

OSM “ways” may be long and heterogeneous. For predictable matching, preprocessing shall convert roads into
**segments** with bounded length and reasonably stable bearing.

Decision:
- Split OSM ways into segments using a deterministic rule, e.g.:
  - split at intersections/junction nodes, and/or
  - split when cumulative length exceeds a threshold, and/or
  - split when heading changes exceed a threshold
- Each segment becomes the unit of matching and attribution.

This improves parallel-road and junction behaviour and reduces distance-to-polyline complexity.

---

### 3. Store geometry in a compact, deterministic encoding

Geometry shall be stored as an ordered list of coordinates per segment.

Decision:
- Store geometry as:
  - encoded polyline (preferred) OR
  - compact JSON array of `(lat, lon)` pairs (acceptable for early V1 if size remains bounded)
- Regardless of encoding, the representation must be:
  - stable across builds given identical input
  - unambiguous (preserve coordinate order)

The database shall also store a precomputed bounding box per segment to support the R*Tree index.

---

### 4. Normalise speed limits at preprocessing time

To avoid inconsistent parsing and unit handling on-device:

- Preprocessing shall parse `maxspeed` tag values and store:
  - `maxspeed_kph` as an integer (canonical)
  - `maxspeed_raw` for debugging/audit
  - `speed_source` = `osm:maxspeed` (V1)

If parsing fails, `maxspeed_kph` shall be NULL and the segment shall be treated as “Unknown” at runtime.

---

### 5. Include derived metadata to support ADR 0002

Preprocessing shall compute and store:

- `heading_deg` (0–360) representative segment bearing
- `road_class` from `highway=*` (for filtering/debug and future estimation)
- optional:
  - `name` (for debug only)
  - `oneway` (future map matching improvements)

---

### 6. Version and provenance are part of the dataset

Each dataset build shall embed metadata sufficient to identify exactly what it was built from.

Decision:
- Add a `dataset_meta` table containing:
  - `region_id` (e.g. `NI`)
  - OSM source (provider, extract name)
  - extract date/time (or version identifier)
  - preprocessing tool version (git commit hash)
  - schema version
  - build timestamp

The app shall surface dataset version in a debug screen/log output.

---

## Schema (Logical)

### `roads` (segments)
- `id` INTEGER PRIMARY KEY
- `maxspeed_kph` INTEGER NULL
- `maxspeed_raw` TEXT NULL
- `speed_source` TEXT NOT NULL
- `highway` TEXT NOT NULL
- `name` TEXT NULL
- `heading_deg` REAL NULL
- `geom` BLOB/TEXT NOT NULL  (encoded polyline or JSON)
- `min_lat` REAL NOT NULL
- `min_lon` REAL NOT NULL
- `max_lat` REAL NOT NULL
- `max_lon` REAL NOT NULL

### `roads_index` (spatial index)
- SQLite R*Tree virtual table keyed by `id`:
  - `id, minLon, maxLon, minLat, maxLat`

### `dataset_meta`
- `key` TEXT PRIMARY KEY
- `value` TEXT NOT NULL

Notes:
- Bounding boxes are stored in both `roads` and `roads_index` for audit/debug; `roads_index` is used for query.

---

## Preprocessing Pipeline

### Inputs
- OSM PBF extract containing Northern Ireland coverage (and only what is needed)
- Preprocessing configuration:
  - segmentation thresholds
  - eligible highway types
  - maxspeed parsing rules and supported formats

### Steps
1. Load OSM PBF on desktop
2. Filter to eligible `highway=*` ways
3. Extract `maxspeed` and relevant tags
4. Build a node graph to identify intersections
5. Segment ways deterministically into road segments
6. For each segment:
   - compute geometry
   - compute bounding box
   - compute representative heading
   - parse and normalise `maxspeed`
7. Write to SQLite:
   - populate `roads`
   - build/populate `roads_index`
   - populate `dataset_meta`
8. Validate:
   - schema checks
   - index integrity
   - basic coverage statistics (unknown rate, segment counts)

### Outputs
- `ni_roads.sqlite` (or similar) plus a build manifest/log

---

## Alternatives Considered

### A. Ship raw PBF and parse on-device
**Rejected**
- Slow startup, heavy CPU/memory, complex Android packaging, unpredictable performance.

### B. Use GeoPackage / shapefiles
**Rejected**
- More friction than SQLite for app queries, more moving parts, less control over indexing strategy.

### C. Store full OSM ways without segmentation
**Rejected**
- Poor matching behaviour at junctions and curved roads; difficult to compute stable bearings.

### D. Store only bounding boxes (no geometry)
**Rejected**
- Insufficient for accurate distance-to-road calculations and stable matching.

---

## Consequences

### Positive
- Fast queries and low runtime overhead
- Repeatable builds with strong provenance
- Geometry and metadata optimised for deterministic matching
- Clear foundation for future enrichment (conditional limits, sign fusion)

### Negative / Trade-offs
- Requires maintaining a preprocessing toolchain
- Dataset updates require rebuilding and redistribution
- Segmentation strategy impacts matching quality and dataset size; must be tuned carefully

---

## Validation and Success Measures

The dataset and pipeline are considered acceptable when:

- Database queries return candidates within latency budgets on target Android hardware
- Segment selection performance meets ADR 0002 validation targets
- Build is reproducible: identical input extract + config yields identical output DB (byte-for-byte where feasible)
- Coverage metrics for NI are known and tracked (e.g. % segments with `maxspeed_kph`)

---

## Future Considerations

- Add support for `maxspeed:conditional` in preprocessing (deferred)
- Add ROI/GB region packs and boundary filtering
- Add compression strategies for geometry (polyline encoding, dictionary tables)
- Introduce multi-level roads/elevation handling (if required)
- Add integration of official restriction feeds (V2+), tracked by new ADRs