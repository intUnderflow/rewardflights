# Aer Lingus — data fields

Award (Avios) seat availability for Aer Lingus, across the AerClub reward network.

## Layout

```
airlines/aer-lingus/
  FIELDS.md                     ← this file
  data/
    <ORIGIN>-<DEST>/            ← one directory per directional METRO route (IATA metropolitan
                                   codes), e.g. DUB-NYC, DUB-BOS, NYC-DUB
      <YYYY-MM-DD>.json         ← one file per departure date that has award availability
```

A route is **directional**: `DUB-NYC` (outbound) and `NYC-DUB` (inbound) are separate directories.

Routes are keyed by **IATA metropolitan / city codes** (`NYC` = JFK + EWR, `LON` = all London
airports, `CHI` = ORD + MDW, …), the same convention British Airways uses here, so a city pair is
comparable across airlines. Where a city has a single relevant airport the airport code is used
directly (`DUB`, `ORK`, `SNN`, `BOS`). Exact airports, flight numbers and per-cabin detail appear
inside each date file's `flights` array.

**A file exists for a date iff that date has award availability** (in at least one cabin). Each file
is re-committed only when its availability changes, so **the git history of each file is the
time-series** — `git log -p <file>` shows how that route + departure date's availability changed
over time.

## File schema

```jsonc
{
  // airline / origin / destination / departureDate are encoded in the PATH (see Layout above),
  // so they are NOT repeated inside the file — the file carries only the availability payload.
  "cabinsAvailable": ["M", "C"],   // cabins with award availability on this date (see Cabin codes)
  "flights": [ Flight, ... ]       // OPTIONAL — per-flight (airport-level) detail
}
```

- **`cabinsAvailable`** — always present. The cabins that have award availability on this date.
- **`flights`** — optional, **additive**. **Absent** means the flight-by-flight breakdown isn't
  available for this date yet — the date is still available (see `cabinsAvailable`), only the
  per-flight detail is pending. **Present** means the full per-flight detail below is included.

### Cabin codes

| Code | Cabin |
|------|-------|
| `M`  | Economy |
| `W`  | Premium Economy |
| `C`  | Business |

Aer Lingus has no First cabin; the `F` code used by other airlines here does not appear.

### Flight object

```jsonc
{
  "flightNumbers": ["EI107"],    // marketing flight number(s); >1 => connecting itinerary, in order
  "carriers": ["EI"],            // operating carrier IATA code(s), deduped
  "via": [],                     // intermediate airport(s) for connections, e.g. ["DUB"]; [] = non-stop
  "depart": "16:45",             // scheduled departure, local 24h HH:MM (first segment)
  "arrive": "19:20",             // scheduled arrival, local 24h HH:MM (last segment)
  "peak": "off-peak",            // Avios pricing band for the date: "off-peak" | "peak"
  "seats": { "M": 7, "C": 1 },   // reward seats available per cabin
  "avios": { "M": 13000, "C": 50000 }  // Avios price per person per cabin, for this flight
}
```

- **`seats`** — reward seats available per cabin. A cabin may appear with `0` when the cabin exists
  on the flight but has no reward availability; `cabinsAvailable` above lists only cabins with at
  least one seat somewhere on the date.
- **`avios`** — Avios price per person for this flight, per cabin. **Aer Lingus only**: other
  airlines in this dataset do not currently publish an award price, so this field is specific to
  `airlines/aer-lingus/`. For a connecting itinerary the value is the itinerary total.
- For connecting itineraries, `seats` is the **tightest** segment (the bookable limit) and `avios`
  is the total for the journey.
