# ADR 0001: Offline, Map-Based Speed Limit Determination (V1)

## Status
Accepted

## Date
2025-04-01

## Context

The project aims to display the most likely legal speed limit for the road a vehicle is travelling on, starting with populated areas within Northern Ireland.

Key constraints and drivers:

- The system must function **fully offline** in V1
- Initial implementation must be **map-based**, not vision-based
- Data must be **locally stored** and efficiently queryable on an Android tablet
- Behaviour must be **deterministic**, testable, and explainable
- Incorrect or uncertain data must fail **explicitly** (e.g. “Unknown”), not silently

OpenStreetMap (OSM) provides speed-limit information via `maxspeed` tags on road segments but is:
- incomplete in places, especially Northern Ireland,
- not authoritative for temporary restrictions,
- heterogeneous in tagging quality.

Despite this, OSM is the most viable open dataset for offline, regional coverage, where cost is a mitigating factor.

---

## Decision

### 1. Use OpenStreetMap as the primary data source (V1)

- Speed limits shall be sourced from OSM `maxspeed` tags
- Only explicitly tagged speed limits shall be treated as authoritative
- No attempt shall be made to infer temporary or variable limits in V1

---

### 2. Preprocess OSM data offline into a local database

Raw OSM PBF files shall **not** be parsed on-device.

Instead:
- A desktop preprocessing step will extract:
  - road geometries (`highway=*`)
  - explicit `maxspeed` values
  - minimal metadata required for matching
- Data will be written into a **SQLite database**
- A spatial index (R*Tree) will be used for proximity queries

This reduces runtime complexity, improves performance, and allows deterministic behaviour.

---

### 3. Perform deterministic map matching at runtime

At runtime:
- GPS position and (if available) heading shall be used
- Candidate road segments shall be selected via spatial query
- The best match shall be chosen using:
  - distance to road geometry
  - heading alignment
- No probabilistic or ML-based matching shall be used in V1

This ensures repeatability and testability.

---

### 4. Treat uncertainty explicitly

If:
- no suitable road match is found, or
- the matched road has no explicit `maxspeed`

then:
- the system shall report **“Unknown”**
- optional estimation may be introduced later, but must be clearly labelled

The system shall never silently guess a limit.

---

### 5. Keep core logic platform-independent

- Speed-limit lookup and map-matching logic shall be written as a platform-agnostic core
- Android-specific code shall be limited to:
  - GPS input
  - UI rendering
  - lifecycle management
- Core logic shall be unit-testable outside Android

---

## Alternatives Considered

### A. Querying online APIs (e.g. Overpass) at runtime
**Rejected**  
Requires connectivity, introduces latency and rate limits, and breaks offline-first requirements.

---

### B. Parsing OSM PBF files directly on device
**Rejected**  
Increases binary size, startup time, memory usage, and implementation complexity on Android.

---

### C. Using proprietary or paid map datasets
**Rejected**  
Conflicts with open-data goals, limits transparency, and complicates redistribution.

---

### D. Inferring speed limits from road type and lighting
**Deferred**  
May be introduced as an explicitly labelled estimate, but is not suitable as a default authoritative source.

---

### E. ML-based map matching
**Rejected (V1)**  
Adds complexity, reduces explainability, and complicates deterministic testing.

---

## Consequences

### Positive
- Fully offline operation
- Deterministic, reproducible behaviour
- Clear failure modes
- Straightforward unit and integration testing
- Clean foundation for later vision-based enhancements

### Negative / Trade-offs
- Incomplete coverage where OSM lacks `maxspeed`
- No handling of temporary or variable limits
- Requires a preprocessing pipeline
- Map updates require rebuilding and redistributing the dataset

---

## Future Considerations

- Additional ADRs may supersede this decision when:
  - camera-based sign recognition is introduced
  - live or semi-live restriction feeds are integrated
  - additional regions are supported

Future ADRs should explicitly state whether they **extend** or **replace** this approach.
