Assumptions baked in (explicit so they can be challenged later):

* **Geography:** Northern Ireland only (initial release)
* **Platform:** Android tablet
* **Connectivity:** Fully offline for V1
* **Purpose:** Informational driver aid (not enforcement, not navigation)
* **Primary data source (V1):** Preprocessed OpenStreetMap extract

---

# Product Goal (one sentence)

> Display the most likely current legal speed limit for the road the vehicle is travelling on, with clear indication of data source and confidence.

---

# MUST HAVE (V1 – required for a usable product)

These are **non-negotiable**. If any fail, the product is not fit for purpose.

## M1. Offline speed-limit lookup from map data

**Description**
The system shall determine the applicable speed limit for the current road segment using locally stored map data, without requiring network connectivity.

**Success criteria**

* Works with **all network interfaces disabled**
* Returns a speed limit for ≥ **85%** of driven distance on A, B, and unclassified roads in NI test routes
* Lookup latency ≤ **100 ms** per GPS update on target hardware
* No runtime parsing of raw OSM PBF on device
* Will compare with [Waze](https://www.waze.com/) speed limit results, running on iPhone 13 (latest iOS 26) as the de-facto standard. No system is 100% correct but will treat Waze as ground truth and consider the differences in results where they arise.

---

## M2. Deterministic map matching

**Description**
The system shall consistently match GPS position and heading to the correct road segment, including at junctions and parallel roads.

**Success criteria**

* Correct road selected ≥ **95%** of the time in junction test cases
* Heading is used to disambiguate parallel/overlapping roads
* Same GPS trace always produces the same road match (no randomness)

---

## M3. Correct interpretation of OSM speed tags

**Description**
The system shall correctly interpret OSM `maxspeed` values, including unit handling.

**Success criteria**

* Correct handling of:

  * numeric values (e.g. `50`)
  * explicit units (e.g. `30 mph`)
* Unit tests cover ≥ **90%** of observed NI `maxspeed` tag formats
* Incorrect/unparseable tags fail safely (no crash, no bogus value)

---

## M4. Explicit “unknown” handling

**Description**
When a speed limit cannot be confidently determined, the system shall explicitly indicate this state.

**Success criteria**

* UI clearly shows **“Unknown”** rather than guessing
* No inferred defaults are presented as factual limits
* Unknown state is logged with reason (e.g. “no maxspeed tag”)

---

## M5. Clear data-source attribution

**Description**
The system shall always indicate where the displayed speed limit came from.

**Success criteria**

* UI labels clearly distinguish:

  * “Map data”
  * “Estimated”
  * (future) “Detected from sign”
* Source attribution is visible without navigating menus
* Source is also emitted in logs/telemetry (if enabled)

---

## M6. Stable, readable driver UI

**Description**
The displayed speed limit shall be readable at a glance and not flicker excessively.

**Success criteria**

* Speed limit text readable in daylight at arm’s length
* No more than **1 change per 3 seconds** unless the road actually changes
* Transitions between limits are visually smooth (no flashing)

---

## M7. Fully testable core logic

**Description**
Core logic shall be independently testable outside Android.

**Success criteria**

* ≥ **80% unit test coverage** on map matching + speed extraction
* No Android dependencies in core speed-limit logic
* Deterministic tests for junction and edge-case scenarios

---

# SHOULD HAVE (strongly recommended, but V1 can ship without them)

These materially improve correctness and credibility.

## S1. Speed-limit confidence scoring

**Description**
The system should compute a confidence score for the displayed limit.

**Success criteria**

* Confidence derived from distance to road, heading alignment, tag quality
* UI reflects confidence qualitatively (e.g. High / Medium / Low)
* Low confidence never presented as authoritative

---

## S2. Road-type-based estimation (clearly marked)

**Description**
Where `maxspeed` is missing, the system should optionally provide an estimated limit based on road classification.

**Success criteria**

* Estimation can be disabled entirely
* Estimated limits are visually distinct from signed limits
* Rules are documented and unit tested
* Never overrides an explicit `maxspeed` tag

---

## S3. Robust GPS noise handling

**Description**
The system should tolerate poor GPS accuracy without erratic behaviour.

**Success criteria**

* Stable output with simulated ±15 m GPS jitter
* No rapid limit switching when stationary
* Graceful degradation when heading is unavailable

---

## S4. Efficient on-device storage

**Description**
Map data storage should be compact and performant.

**Success criteria**

* NI dataset ≤ **200 MB** on device
* Cold start time ≤ **3 seconds**
* Spatial index queries scale logarithmically

---

## S5. Trace and replay capability

**Description**
The system should allow recorded GPS traces to be replayed for testing.

**Success criteria**

* Offline replay produces identical outputs to live mode
* Replays usable in automated tests
* Trace format documented

---

# COULD HAVE (useful, but optional for early releases)

These improve usability, extensibility, or future V2 work.

## C1. Speed-limit change notifications

**Description**
The system could optionally notify the user when the limit changes.

**Success criteria**

* Can be disabled
* No audio/visual spam at complex junctions
* Delay configurable

---

## C2. Visual road-matching debug overlay

**Description**
A developer/debug mode could show the matched road geometry.

**Success criteria**

* Disabled by default
* Shows candidate roads + chosen segment
* Zero performance impact when disabled

---

## C3. Multi-region support architecture

**Description**
The system could support additional regions via data packs.

**Success criteria**

* Region boundaries cleanly separable
* No code changes required to add England/Scotland/ROI
* Region selection explicit

---

## C4. Telemetry hooks (privacy-first)

**Description**
The system could emit anonymised correctness metrics.

**Success criteria**

* Opt-in only
* No raw GPS coordinates transmitted
* Useful for measuring unknown/estimated rates

---

# WON’T HAVE (explicitly out of scope for now)

These are **not** to be implemented in this phase.

## W1. Real-time temporary speed limits

* No live road-works feeds
* No TRO ingestion
* No variable gantry logic

## W2. Camera-based sign recognition (V2 only)

* No camera access
* No ML inference
* No sign detection heuristics

## W3. Navigation or routing

* No turn-by-turn
* No map rendering beyond minimal UI
* No traffic awareness

## W4. Legal or enforcement guarantees

* No claims of legal correctness
* No compliance certification
* Informational use only

---

# Definition of V1 “Done”

V1 is considered complete when:

* All **MUST** requirements pass their success criteria
* No **SHOULD** requirement blocks correctness
* System operates offline for **at least 1 hour** of continuous driving without degradation
* Output is stable, explainable, and test-reproducible