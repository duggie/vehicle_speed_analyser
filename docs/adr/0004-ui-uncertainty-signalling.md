```markdown
# ADR 0004: UI Presentation and Uncertainty Signalling (V1)

## Status
Accepted

## Date
2025-04-02

## Context

The system displays speed limit information to a driver in real time.
Incorrect or ambiguous presentation creates safety risk, user distrust, and legal exposure.

Key challenges:

- Speed limits may be missing, stale, or ambiguous
- Map matching may be uncertain near junctions or under poor GPS conditions
- Frequent changes (“flicker”) reduce usability and confidence
- Drivers must immediately understand:
  - what the system believes the limit is
  - how confident the system is
  - where the information came from

The UI must therefore be:
- glanceable
- stable
- explicit about uncertainty
- conservative in its claims

---

## Decision

### 1. Display a single primary speed limit value

At any moment, the UI shall display at most one speed limit value.

- No lists
- No ranges
- No competing values

This avoids cognitive overload and conflicting signals.

---

### 2. Explicit “Unknown” is a first-class UI state

If the system cannot confidently determine a speed limit, the UI shall display:

> **UNKNOWN**

rather than hiding the value or guessing.

This state is as valid as a numeric limit.

---

### 3. Source attribution is always visible

The UI shall always indicate the source of the displayed value.

For V1, valid sources are:
- **Map data**
- **Estimated** (if enabled)
- **Unknown**

Source text shall be visible without additional interaction.

---

### 4. Uncertainty is communicated qualitatively, not numerically

Confidence shall be communicated using a small, qualitative indicator.

Decision:
- Use a discrete set such as:
  - High
  - Medium
  - Low

or equivalent iconography.

Numeric confidence percentages are explicitly rejected to avoid false precision.

---

### 5. Conservative visual hierarchy

The UI shall visually prioritise safety and clarity.

Rules:
- Numeric speed limit is the largest element
- Unit (`mph` or `km/h`) is always shown
- Source and confidence are secondary but readable
- Unknown or low-confidence states shall not visually resemble authoritative values

---

### 6. Stability over immediacy

The UI shall prioritise stability over instant reaction.

Decision:
- Apply debounce/hysteresis before updating the displayed value
- Avoid rapid toggling between values at junctions
- A small, intentional delay is acceptable if it reduces flicker

---

### 7. No implied legal authority

The UI shall not imply that the displayed limit is legally authoritative.

Rules:
- No language such as “legal”, “official”, or “enforced”
- No colours or styling that mimic official road signage
- Disclaimer wording exists outside the main driving view

---

## UI States (V1)

The UI must support at least the following states:

### A. Confident, map-derived limit
- Large numeric value
- Unit displayed
- Source: “Map data”
- Confidence: High or Medium

### B. Uncertain, map-derived limit
- Numeric value shown
- Confidence: Low
- Visual cue indicating uncertainty

### C. Unknown
- Text: “Unknown”
- Source: “Map data”
- Confidence indicator absent or explicitly “Unknown”

### D. Transitional (optional)
- Briefly shown during road transitions
- Should not distract or animate excessively

---

## Visual Design Constraints

- Readable in daylight at arm’s length
- High contrast, colour-blind safe palette
- No flashing, pulsing, or aggressive animations
- Works in portrait and landscape orientations

Exact colour values and typography are implementation details and intentionally not fixed here.

---

## Alternatives Considered

### A. Always show an estimated limit if exact is missing
**Rejected**  
Encourages overtrust and hides uncertainty.

---

### B. Show multiple candidate limits
**Rejected**  
Increases cognitive load and decision-making burden while driving.

---

### C. Numeric confidence scores (e.g. 87%)
**Rejected**  
Creates false precision and is poorly understood by users.

---

### D. Mimic official road sign styling
**Rejected**  
Creates legal and ethical risk and may mislead users.

---

## Consequences

### Positive
- Clear communication of uncertainty
- Reduced user overreliance
- Stable, glanceable interface
- Lower legal and reputational risk

### Negative / Trade-offs
- Conservative behaviour may show “Unknown” more often
- Less visually engaging than consumer navigation apps
- Requires careful tuning of debounce thresholds

---

## Validation and Success Measures

The UI approach is considered acceptable when:

- Users correctly identify whether the displayed limit is certain or uncertain
- No rapid flicker occurs during normal driving
- Unknown state is understood and not perceived as a bug
- Transitions are calm and predictable

Validation may include:
- Controlled drive tests
- Simulated GPS jitter playback
- Informal usability testing

---

## Future Considerations

- Introduce vision-based source (“Detected from sign”) in V2
- Allow user to tap for detailed debug/explanation view
- Region-specific units and terminology
- Night mode considerations (future ADR if required)