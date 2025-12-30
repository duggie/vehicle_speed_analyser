# Speed Limit Display (Northern Ireland)

An offline, map-based system that displays the most likely legal speed limit for the road a vehicle is travelling on, starting with Northern Ireland.

This project is intentionally developed in stages:
- **V1:** Offline, map-based speed limits using preprocessed OpenStreetMap data
- **V2:** Camera-based speed-limit sign recognition for temporary and variable limits

The system is designed as an **informational driver aid**, not a navigation or enforcement tool.

---

## Project Goals

- Display the current road speed limit clearly and reliably
- Work fully offline in V1
- Prefer correctness and explainability over guesswork
- Be testable, deterministic, and auditable
- Clearly indicate when the speed limit is unknown or estimated

---

## Scope (Initial Release)

### In Scope
- Northern Ireland only
- Offline operation
- Map-based speed limits from OpenStreetMap
- Simple, glanceable driver UI
- Deterministic map matching using GPS position and heading
- Explicit handling of unknown or uncertain limits

### Out of Scope
- Temporary or road-works speed limits
- Variable gantry limits
- Camera or vision processing
- Turn-by-turn navigation
- Traffic data
- Legal or enforcement guarantees

---

## Data Source

- **OpenStreetMap (OSM)**  
  Speed limits are sourced from `maxspeed` tags on road segments and preprocessed into a local, query-efficient database.

OSM data is treated as **best-effort**. Where no explicit speed limit is present, the system may report “Unknown” or (optionally) an estimated value that is clearly labelled as such.

---

## High-Level Architecture (V1)

1. **Preprocessing (desktop)**
   - Extract road geometry and speed limits from OSM
   - Generate a compact, indexed local database

2. **On-device runtime**
   - Receive GPS position and heading
   - Match location to the most likely road segment
   - Display the associated speed limit with source attribution

Core speed-limit logic is platform-independent and testable outside Android.

---

## Non-Goals

This project does **not** attempt to:
- Replace vehicle speed-limit systems
- Provide legally authoritative speed limits
- Infer unsigned or temporary restrictions as fact

If a limit cannot be confidently determined, it will be shown as **Unknown**.

---

## Development Principles

- Offline first
- Deterministic behaviour
- Clear failure modes
- Explicit uncertainty
- High test coverage for core logic
- Minimal UI, focused on clarity

---

## Planned Roadmap

### V1
- Offline map-based speed limits (Northern Ireland)
- Deterministic map matching
- Clear source and confidence signalling

### V2 (future)
- On-device camera-based sign recognition
- Fusion of map and vision sources
- Handling of temporary and variable limits

---

## Disclaimer

This software is provided for informational purposes only.  
Drivers remain responsible for observing and complying with all posted road signs and applicable laws.

---

## Status

This repository currently contains **no implementation code**.  
The README and requirements define the intended behaviour before development begins.