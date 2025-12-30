# ADR 0002: Deterministic Map Matching and Road Selection Strategy (V1)

## Status
Accepted

## Date
2025-04-01

## Context

V1 must determine the most likely road segment the vehicle is travelling on using offline data, then display that segment’s explicit `maxspeed` (or “Unknown”).

The key challenges:

- GPS error (often 5–15m, sometimes worse)
- Parallel roads (dual carriageways, slip roads, service roads)
- Junction ambiguity (roundabouts, multi-branch intersections)
- Intermittent or unavailable heading (course) data at low speeds
- Need for deterministic, testable, explainable behaviour
- Low latency requirements on Android hardware

We are not doing probabilistic or ML-based matching in V1, and we are not using live map tiles or routing engines.

---

## Decision

### 1. Candidate selection via spatial index (R*Tree bbox query)

At each GPS update, the system shall:
- query the spatial index for candidate road segments within a search window derived from a radius (metres)
- produce a bounded set of candidate segments for scoring

Candidate selection is purely geometric and deterministic.

---

### 2. Scoring based on distance and heading alignment

The system shall score each candidate segment using a deterministic function:

- **Primary factor:** geometric distance from GPS point to the segment polyline
- **Secondary factor:** heading alignment between GPS course and segment bearing (when course is available and valid)

The chosen road is the candidate with the lowest total score (after filtering).

---

### 3. Heading as a filter, not a mandatory requirement

Heading shall be used to reduce false matches but must not prevent matching when it is unavailable.

Rules:
- If GPS course is available and speed exceeds a minimum threshold, apply a heading filter.
- If GPS course is unavailable or speed is below the threshold, score using distance only.

This avoids instability when stationary or moving slowly.

---

### 4. Road selection must be stable over time (hysteresis)

To avoid UI flicker and noisy switching, the matcher shall implement hysteresis:

- Once a segment is selected, the system should continue to select it unless:
  - a different segment is significantly better, or
  - the current segment becomes invalid (e.g. no longer in candidate set)

This creates stable output while still allowing genuine road changes.

---

### 5. Explicit match failure conditions

The system shall return “Unknown” if:
- no candidates are returned by the spatial query, or
- all candidates are rejected by filters, or
- the best candidate’s distance exceeds a configured maximum threshold

Failure reasons shall be surfaced to logs for debugging and test evaluation.

---

## Design Details

### Candidate selection parameters

- `search_radius_m`: initial radius to query candidates (configurable)
- `max_distance_m`: maximum allowed distance from GPS point to the matched segment
- Spatial query uses a bbox approximation; exact distances are computed during scoring.

---

### Segment bearing calculation

Each road segment shall have a representative bearing.

Decision:
- During preprocessing, compute and store a `heading_deg` for each segment (0–360).
- Bearing may be computed as:
  - overall bearing from first→last coordinate, or
  - median of per-subsegment bearings (preferred if stored)

V1 shall accept the simplest approach initially, provided tests cover known edge cases
(e.g. curved segments).

---

### Heading validity and usage

Heading filter applies only when:
- GPS course is available, and
- speed is above `min_speed_for_heading_mps` (configurable), and
- course value is within [0, 360)

Heading delta is computed as the minimal absolute angular difference.

Filter:
- reject candidates whose heading delta exceeds `max_heading_delta_deg` (configurable)

Scoring:
- for remaining candidates, add a weighted term:
  - `score = distance_m + (heading_weight * heading_delta_deg)`

Weights are deterministic constants stored in configuration.

---

### Hysteresis strategy

Maintain the previously matched segment (`prev_id`) and apply:

- Prefer `prev_id` if it remains within candidate set and:
  - its score is within `stickiness_margin` of the best candidate, or
  - the best candidate is only marginally better

Switch only when the alternative is convincingly better
(e.g. exceeds a score improvement threshold).

This is required to prevent oscillation near junctions and parallel roads.

---

## Alternatives Considered

### A. Nearest-road-only (distance-only) matching
**Rejected**  
Fails in parallel-road scenarios and near junctions. Produces unstable results.

---

### B. Full routing-based map matching (HMM / Viterbi)
**Rejected (V1)**  
More accurate but significantly more complex, less explainable, and heavier to implement/test.

---

### C. Use a third-party navigation engine for matching
**Rejected**  
Introduces dependency and reduces control over determinism and testability. Offline setup and licensing may complicate distribution.

---

### D. Use heading as mandatory constraint
**Rejected**  
Heading is often unreliable at low speeds, when stationary, or in poor GPS conditions. Would increase “Unknown” rates unnecessarily.

---

## Consequences

### Positive
- Deterministic, explainable matching
- Works offline with low compute overhead
- Robust against common ambiguity via heading and hysteresis
- Straightforward unit testing via synthetic geometries and GPS traces

### Negative / Trade-offs
- Curved roads may have less precise bearing representation
- Still vulnerable to extreme GPS drift or multi-level roads (bridges) without elevation
- Requires careful parameter tuning (thresholds/weights)

---

## Validation and Success Measures

The matching strategy is considered acceptable when:
- correct segment selection ≥ 95% in a junction-heavy NI test suite
- stable UI output with ≤ 1 spurious switch per 3 seconds in noisy GPS simulations
- deterministic replay: same input trace yields identical output sequence

---

## Implementation Notes (Non-binding)

- Candidate query: SQLite R*Tree bbox search
- Distance: point-to-polyline distance in a local projection (e.g. equirectangular approximation)
- Hysteresis: store `prev_id` and `prev_score`, apply switching logic
- All constants configurable (but shipped with sensible defaults)