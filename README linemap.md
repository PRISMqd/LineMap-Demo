# LineMap — PRISMqd LineLogic Prototype

## Overview
LineMap is a patient-level infusion safety and line management prototype for high-acuity care environments. It is an **assistive** clinical decision-support system — it supports clinician judgment, shows transparent rationale for every alert, and allows override with acknowledgment. It never silently assumes compatibility.

---

## How to Run

### Immediate (no install)
Open `linemap.html` in any modern browser. No server, no build step, no network required.

### Local server (optional, for development)
```bash
python3 -m http.server 8080
# Then open http://localhost:8080/linemap.html
```

---

## Data Loading

All data is **embedded directly in the HTML file** in the `MEDICATIONS`, `PAIR_RULES`, and `POLICY_PROFILE` constants near the top of the `<script>` block.

### To replace with external JSON files (future):
Swap the inline constants with fetch calls:
```js
const MEDICATIONS = await fetch('./medications_full.json').then(r => r.json());
const PAIR_RULES   = await fetch('./pair_rules_full.json').then(r => r.json());
const POLICY_PROFILE = await fetch('./policy_profiles_full.json').then(r => r.json());
```

### Medication record shape (medications_full.json)
```json
{
  "med_id": "van",
  "display_name": "Vancomycin",
  "generic_names": ["vancomycin"],
  "brand_names": ["Vancocin"],
  "abbreviations": ["vanc", "vanco"],
  "aliases": ["vanc", "vanco"],
  "searchable_text": "vancomycin vanc vanco vancocin IV",
  "class": "antibiotic",
  "search_priority": 1,
  "requiresCentral": false,
  "vesicant": false,
  "filterRequired": false,
  "requiresDedicated": false
}
```

### Pair rule shape (pair_rules_full.json)
```json
{
  "med_a": "van",
  "med_b": "pip",
  "sameLumen": "incompatible",
  "ySite": "compatible",
  "separateLumen": "compatible",
  "notes": "Compatible Y-site. Incompatible same lumen."
}
```
Compatibility values: `compatible | caution | avoid | incompatible | unknown`

### Policy profile shape (policy_profiles_full.json)
```json
{
  "profileId": "default_icu",
  "centralVesicantRequired": true,
  "peripheralVesicantAllowed": false,
  "peripheralVesicantAllowedWithAck": true,
  "unknownCompatibilityDefault": "caution"
}
```

---

## Architecture

### Single-file, offline-first
- React 18 via CDN (Babel-transpiled JSX, no build)
- All logic is deterministic rule evaluation — no ML, no network calls
- Local state only (React useState)

### Screens
| Screen | Description |
|---|---|
| **Patient List** | All patients with state badges, line burden, quick entry |
| **Patient Workspace** | Body map + line/lumen cards + alert panel + action bar |
| **Add Medication Overlay** | Search-first with engine evaluation + custom entry |
| **Quick Check** | Emergency mode — check compatibility without full patient |
| **Audit Drawer** | All actions logged with type, timestamp, detail |

### Assignment Engine (`runAssignmentEngine`)
Given a patient state + new medication:
1. Checks dedicated/vesicant/filter requirements
2. Evaluates each lumen for primary assignment feasibility
3. Evaluates Y-site capacity where applicable
4. Returns: `safe_capacity_available | unsafe_reconfigurable | unsafe_new_access_recommended | unresolvable_escalate`
5. Provides ranked safe and caution placements with rationale

### Search (`searchMedications`)
Ranking order:
1. Exact alias match
2. Exact generic match
3. Exact brand match
4. Exact abbreviation
5. Starts-with (any token)
6. Substring (searchable_text)

### Compatibility resolution
- Same lumen → `sameLumen` rule field
- Y-site → `ySite` rule field
- Worst rule wins across all pairs
- Missing rule → `unknown` (never assumed safe)

---

## Prototype Logic vs Future Governed Logic

| Component | Prototype (current) | Future / Licensed |
|---|---|---|
| Medication library | 20 common ICU agents, embedded | Full licensed pharmacopeia (Lexicomp, Micromedex, etc.) |
| Pair rules | ~25 common pairs, manually curated | Governed pair database with evidence grades |
| Policy profiles | Single default ICU profile | Institution-specific, role-based, versioned profiles |
| Compatibility defaults | `caution` for unknown | Configurable per institution, per drug class |
| Audit log | In-memory, session only | Persistent, signed, exportable audit trail |
| Override logging | Local state | Countersigned, time-stamped, clinician-identified |
| Body map coordinates | Approximate % positions | Validated anatomical overlays |
| Patient data | Demo data only | EHR integration (FHIR R4) |
| Filter rules | Simplified flags | Full filter compatibility matrix |

---

## Non-Negotiables (implemented)
- ✅ Clinician override always available — no permanent hard blocks
- ✅ Y-site capacity modeled separately from primary lumen capacity
- ✅ `unknown` always shown explicitly — never silently treated as safe
- ✅ Manual entry unrestricted — any line, any lumen count, custom meds
- ✅ Body map: tap = inspect, long-press = create
- ✅ Every alert states what, where, and recommended action
- ✅ No punitive language — uses "not recommended", "separate lumen preferred", "compatibility not verified"
- ✅ Offline-first — no network dependency for core function

---

## Demo Patients

| Patient | Location | Key Features |
|---|---|---|
| Demo Patient A | ICU-4A | Septic shock, vasopressor on central, TPN dedicated PICC, Y-site in use |
| Demo Patient B | ICU-6B | Post-cardiac, amiodarone + vasopressor conflicts, spare PIV |
| Demo Patient C | ICU-2C | Propofol dedicated lumen, fentanyl/midazolam Y-site, cefepime PIV |

---

## Known Prototype Limitations
- Body map coordinates are approximate; not validated to anatomical standard
- Medication library is a representative sample only — clinical decisions must not rely on prototype data
- Audit log is session-only — cleared on page refresh
- No authentication, no multi-user, no real patient data
- Filter compatibility is simplified to flag/size only
- No drag-and-drop reassignment in this iteration

---

*LineMap — PRISMqd LineLogic Division*  
*Prototype v0.1 — For demonstration and development evaluation only. Not for clinical use.*
