# British Airways — data fields

Award (Avios) seat availability for British Airways, across BA's reward network.

## Layout

```
airlines/british-airways/
  FIELDS.md                     ← this file
  data/
    <ORIGIN>-<DEST>/            ← one directory per directional METRO route (IATA metropolitan
                                   codes), e.g. LON-TYO, LON-SIN, LON-BLL
      <YYYY-MM-DD>.json         ← one file per departure date that has award availability
```

A route is **directional**: `LON-TYO` (outbound) and `TYO-LON` (inbound) are separate directories.

Routes are keyed by **IATA metropolitan / city codes** (`LON` = all London airports, `TYO` = Tokyo
Haneda + Narita, `NYC` = New York JFK + EWR, …), so a route covers a whole city pair. Where finer
airport- and flight-level detail is available, it appears inside each date file's optional `flights`
array (see below).

**A file exists for a date iff that date has award availability** (in at least one cabin). Each file is
re-committed only when its availability changes, so **the git history of each file is the time-series** —
`git log -p <file>` shows how that route + departure date's availability changed over time.

## File schema

```jsonc
{
  "airline": "BA",                     // always "BA" in this folder
  "origin": "LON",                     // departure metro (IATA metropolitan / city code)
  "destination": "TYO",                // arrival metro
  "departureDate": "2026-10-15",       // local departure date, ISO YYYY-MM-DD
  "cabinsAvailable": ["M", "W", "C"],  // cabins with award availability on this date (see Cabin codes)
  "flights": [ Flight, ... ]           // OPTIONAL — per-flight (airport-level) detail, present when
                                       //   available for this route + date
}
```

- **`cabinsAvailable`** — always present. The cabins that have award availability on this date.
- **`flights`** — optional, **additive**. **Absent** means the flight-by-flight / airport-level
  breakdown isn't available for this date yet — the date is still available (see `cabinsAvailable`),
  only the per-flight detail is pending. **Present** means the full per-flight detail below is included.

### Cabin codes

| Code | Cabin |
|------|-------|
| `M`  | Economy |
| `W`  | Premium Economy (World Traveller Plus) |
| `C`  | Business (Club World / Club Europe) |
| `F`  | First |

### Flight object

```jsonc
{
  "flightNumbers": ["BA0007"],   // marketing flight number(s); >1 => connecting itinerary, in order
  "carriers": ["BA"],            // operating carrier IATA code(s), deduped
  "via": [],                     // intermediate airport(s) for connections, e.g. ["DUB"]; [] = non-stop
  "depart": "13:00",             // scheduled departure, local 24h HH:MM (first segment)
  "arrive": "08:55",             // scheduled arrival, local 24h HH:MM (last segment)
  "peak": "off-peak",            // BA Avios pricing band for the date: "off-peak" | "peak" | null (unknown)
  "rewardFlightSaver": true,     // true if this flight offers Reward Flight Saver (reduced cash surcharge)
  "seats": { "W": 6, "C": 1, "F": 0 }  // seats available per cabin
}
```
