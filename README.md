# Campus Equipment Finder

A static website that lets anyone search for a piece of lab equipment and see which
room it's in, highlighted on a map, with the in-charge's contact details. No server,
no database — it runs entirely in the browser and is free to host on GitHub Pages.

## What to upload

Keep this folder structure in your GitHub repo:

| File | What it is | Do you edit it? |
|------|------------|-----------------|
| `index.html` | The whole website (campus base map is baked in) | Rarely |
| `data/csv/equipment.csv` | `Building, Room, Equipment, PO_Link` — one row per item | **Yes, often** |
| `data/csv/contacts.csv` | `Building, Room, In-Charge, Mobile, Email` — one row per room | When a person changes |
| `data/geojson/rooms_updated.geojson` | Room locations exported from QGIS | When rooms change |
| `data/geojson/buildings.geojson` | Building footprints (drawn + labelled, not searched) | When buildings change |
| `logo/logo.png` | College logo loaded by the website | Replace for each college |
| `data/config.json` | College name, centre/tool name, app title, logo path | Edit for each college |
| `README.md` | This guide | No |

The folder also contains extra spatial/source files kept for reference:
- the campus background layers in `data/geojson/` (`greenery.geojson`, `roads.geojson`, `ground.geojson`,
  `foodcourt.geojson`, etc.) — these are your editable masters.
- the rollout helper `data/geojson/rooms_template.geojson` and `DATA_COLLECTION_GUIDE.md` for campus-wide data collection.

The data files are joined by **Building + Room**. This prevents rooms on the same floor in different buildings from mixing together.

`data/geojson/buildings.geojson` is drawn as tan footprints labelled with each building's name, read from
the `Building Name` attribute (the site also recognises `Name`, `Building`, `Block`, etc.). A
few polygons with a blank name fall back to `Building <Id>`. It is **not** searched. Export it
from QGIS as **EPSG:4326**; your file came in as UTM and was reprojected for the web map.

## Campus base map (replaces OSM)

The OpenStreetMap background has been removed. Instead, the campus is drawn from your own
vector layers — greenery (green), ground (sand), cricket ground (turf), running track,
roads and bus bay (grey), food court (amber), plaza, and the buildings. These layers are
**baked into `index.html`**, so the page is
self-contained and needs no internet tiles. They're context only and aren't searched.

The reprojected source files (`data/geojson/greenery.geojson`, `data/geojson/roads.geojson`, etc.) are included as editable masters. Because they're baked into the page, changing one means
re-generating `index.html` — send me the updated layer and I'll rebuild. (Your day-to-day
`data/csv/equipment.csv` edits are unaffected; those are still read live.) All of these layers came in
as UTM (EPSG:32643) and were reprojected to EPSG:4326 for the web map; export future layers
in EPSG:4326 to skip that step.

## Adding new equipment (the everyday task)

Open `data/csv/equipment.csv`, add a line, commit it. That's the whole workflow.

```
Building,Room,Equipment,Last_Calibrated,Next_Calibration_Due,Photo_Link,SOP_Link,PO_Link,Purchase_Date,Cost,Vendor,Warranty_Until,Funding_Source
JC BOSE Block,JC305,Thermal Camera,2025-08-12,2026-08-12,https://drive.google.com/file/d/FILE_ID/view,https://drive.google.com/file/d/FILE_ID/view,https://drive.google.com/file/d/FILE_ID/view,2024-03-15,185000,Vendor Name,2027-03-14,Department Budget
```

- `PO_Link` is optional. Paste the Google Drive share link for that item's purchase
  order and a **PO** button appears next to it in the room popup; leave it blank for none.
  Make sure the Drive file's sharing is set so your users can open it.
- The website re-reads the CSV every time it loads, so a new item is searchable
  immediately after you push the change. You never touch `index.html`.
- The **"Last updated"** date in the top ribbon is read automatically from the file's
  modified date when served over GitHub Pages — no manual editing.

## Put it on GitHub Pages

1. Create a repository and upload the full folder structure to the top level.
2. Repo **Settings → Pages**.
3. Under *Build and deployment*, set **Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Save. After a minute your site is live at `https://<your-username>.github.io/<repo>/`.

## Previewing on your own computer

Because the page loads the CSV/GeoJSON files, you can't just double-click `index.html`
(browsers block file reads from `file://`). Run a tiny local server instead:

```
cd the-folder-with-these-files
python -m http.server 8000
```

Then open `http://localhost:8000`. If you ever see the amber **"Demo mode"** banner,
it means the data files weren't reached — you opened the file directly instead of
through a server.

## How search works

The page is search-first: nothing is shown until you search, so the campus map stays clean.

- **Equipment** — type a name (e.g. `oscilloscope`). Every room that has it is listed and
  pulses on the map; click one to see its equipment, the in-charge, and a **PO** link per item.
- **Faculty** — type an in-charge's name to pull up every room and instrument that person
  manages across the campus.
- **Room** — type a room code (e.g. `JC305`) to jump straight to it.

Results from another building or floor appear with a small building/floor tag; clicking one switches the selectors automatically.

## Your location on the map

A locate button sits at the bottom-right of the map. Tap it to show your position as a blue
dot (with an accuracy ring) and recenter on yourself; the page also makes a quiet attempt to
find you on load without moving the campus view. This needs the browser's location permission
and a secure origin — it works on **GitHub Pages (HTTPS)** and on `localhost`, but not when
the file is opened directly from disk.

## Things to fix / know

- **`JC320` has a blank floor** in the GeoJSON. The site currently shows blank-floor rooms
  on every floor as a fallback. In QGIS, set its `Floor` to `3` and re-export to fix it.
- **Rooms are points, not outlines.** "Highlight" is an animated locator ring on the room's
  point. If you later digitize real room polygons, the site can fill the shape instead with
  only a small change.
- **Adding more buildings/floors:** export rooms into the same GeoJSON with `Building` and `Floor`
  filled. The building and floor dropdowns build themselves from the data, so no code
  change is needed.
- The contact names, phone numbers, and emails in `data/csv/contacts.csv` are **placeholders** —
  replace them with the real in-charge details.


## Replicating for another college

To reuse this tool for another college, keep the same file names and replace only the contents:

- Upload the new college logo as `logo/logo.png`. The website fetches the logo from this folder automatically.
- Edit `data/config.json` to change the college name, centre/tool name, app title, and logo path.
- Update inventory data in `data/csv/equipment.csv`.
- Update room contact data in `data/csv/contacts.csv`.
- Replace campus/room/building spatial layers in `data/geojson/`.

This avoids editing `index.html` and `admin.html` for each college.


## Location-aware zoom

When a user selects an equipment point or result row, the map now avoids excessive zoom. If the user location is available, it fits both the user position and selected equipment location within the visible map area. The location button on the map can be used to refresh/show the current position.
