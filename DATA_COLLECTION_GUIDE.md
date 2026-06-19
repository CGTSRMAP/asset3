# Data Collection Guide — Campus Equipment Finder

You're filling three things. They're tied together by **Building + Room**.
Get those right and everything else just works across multiple buildings.

```
   data/geojson/rooms_updated.geojson   →  WHERE each room is (a point on the map) + which floor/building
   data/csv/equipment.csv           →  WHAT equipment is in each room
   data/csv/contacts.csv            →  WHO to contact for each room
            ↑ all three reference the same Building + Room, e.g. JC BOSE Block + JC301
```

---

## The three golden rules

1. **Building must be filled wherever possible.**
   The app now filters as **Building → Floor**, so `Floor 3` in one block will not mix with `Floor 3` in another block.
   
2. **The room code must be spelled identically everywhere** — same letters, same case, no
   extra spaces. `JC301` in the GeoJSON must be `JC301` (not `jc301` or `JC 301`) in the CSVs.

3. **Every room must have a Floor number.** Don't leave it blank (this is the one thing that
   was wrong with `JC320`). Use a plain integer: `1`, `2`, `3`…

---

## 1. `data/geojson/rooms_updated.geojson` (you build this in QGIS)

One point per room. The site reads two fields from each point; a third is recommended now
that you're going campus-wide.

| Field (exact spelling) | Required? | Type | Notes |
|------------------------|-----------|------|-------|
| `Room` | **Required** | text | The unique room code. The join key. |
| `Floor` | **Required** | integer | Plain number, filled for every room. |
| `Building` | **Required for campus-wide use** | text | e.g. `JC BOSE Block`, `X Lab`. Drives the building selector. |
| `id` | Optional | integer | Auto row number; harmless. |

Other rules for the GeoJSON:
- **Coordinate system must be WGS84 / EPSG:4326** (longitude, latitude). In QGIS, when you
  *Export → Save Features As → GeoJSON*, set **CRS = EPSG:4326** in that dialog. If the layer
  is in a metric/projected CRS, the export step will reproject it for you.
- Geometry type **Point** (the current site highlights a point with a locator pulse). If you
  later digitise room **polygons** instead, keep the same `Room`/`Floor`/`Building` fields and
  tell me — the site can fill the shape instead of pulsing a dot.
- The `Equipement` field that QGIS created earlier is not used by the site; leave it or delete
  it, it doesn't matter.

`rooms_template.geojson` shows the exact structure with two example rooms in two buildings.

## 2. `data/csv/equipment.csv`

One row per piece of equipment — so a room with 8 items has 8 rows, all sharing
the same `Building` and `Room`. To add equipment later, you just add a row.

```
Building,Room,Equipment
JC BOSE Block,JC301,Oscilloscope
JC BOSE Block,JC301,Digital Multimeter
Library Block,LIB101,Barcode Scanner
```

Optional extra columns you can add **now** if you want richer records (Quantity, model, asset
tag, status). The site won't display them yet, but collecting them now saves re-surveying
later, and I can wire them into the room popup whenever you ask:

```
Building,Room,Equipment,Quantity,Make_Model,Asset_Tag,Status
JC BOSE Block,JC301,Oscilloscope,2,Tektronix TBS1052,JC-OSC-014,Working
```

## 3. `data/csv/contacts.csv`

One row per room. Replace the placeholders with the real in-charge.

```
Building,Room,In-Charge,Mobile,Email
JC BOSE Block,JC301,Dr. A. Sharma,+91-90000-00001,incharge.jc301@example.edu
```

If many rooms in a building share the same in-charge, you'll repeat that person across several
rows — that's fine. If you'd rather list one contact per building or per lab instead of per
room, that's a small design change; tell me and I'll switch the key.

---

## Campus-wide building and floor menu

The site now filters as **Building → Floor**. First select the building, then select the floor
within that building. This keeps the map readable and prevents `Floor 3` from multiple
buildings appearing together.

## Quick checklist before you hand me the files

- [ ] `Building` is filled consistently in the room GeoJSON and CSVs.
- [ ] Room codes match exactly between the GeoJSON and both CSVs (case + spacing).
- [ ] Every room has a `Floor` number (none blank).
- [ ] GeoJSON exported as **EPSG:4326**.
- [ ] `Building` filled in for every room.
- [ ] Example rows deleted from the CSVs and replaced with real data.
