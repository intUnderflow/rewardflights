# rewardflights

Open, version‑controlled **award (frequent‑flyer) seat availability** data.

Each airline lives in its own folder under [`airlines/`](airlines/). Each availability file is
re‑committed only when its availability changes, so **git history is a free time‑series** —
`git log -p <file>` shows exactly how that route + date's award availability changed over time.

## Airlines

| Airline | Folder | Fields |
|---------|--------|--------|
| British Airways | [`airlines/british-airways/`](airlines/british-airways/) | [`FIELDS.md`](airlines/british-airways/FIELDS.md) |

More airlines will be added under `airlines/` over time; each is self‑contained with its own
`FIELDS.md` documenting its data.

## Using the data

Data files are plain JSON. For British Airways, one file per directional **metro route** + departure
date:

```
airlines/british-airways/data/LON-TYO/2026-10-15.json
```

See the airline's `FIELDS.md` for the exact schema. In short, each file lists the cabins with award
availability on that date, plus — where available — an optional per‑flight breakdown (flight numbers,
times, and per‑cabin seat counts).

- A file **exists** for a date only when that date has award availability; a **missing** file means no
  availability (or not covered yet), not an error.
- Files are re‑committed only when availability changes — use `git log` / `git blame` for the history
  and to see how fresh the data is.

## License

Licensed under the [Open Database License (ODbL) v1.0](LICENSE) — free to copy, share, and adapt, provided any **adapted database** you publicly distribute is also licensed under ODbL (share-alike), with
attribution. Provided **as‑is**, with no warranty; see the license for the full disclaimer of
warranties and limitation of liability.

## Disclaimer

Availability is a point‑in‑time snapshot provided **as‑is** with no guarantee of accuracy or
bookability, and with no affiliation to or endorsement by any airline.
