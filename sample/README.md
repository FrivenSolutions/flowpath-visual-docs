# Sample report

`flow-map-sample.pbix` — the report Microsoft asks for with a Power BI visual submission, and
the one linked from the user guide.

It must work with no network access: import the data rather than linking to a source, so the
file opens and renders for someone who has never seen the CSV.

The data it is built from lives in the visual's own repository:

| File | Purpose |
|---|---|
| `sample-data.csv` | 61 rows of coordinate-based flows, including 4 multi-leg routes and 9 carrying a transit loss |
| `sample-data-locations.csv` | 48 rows using the `City\|State\|Country` fields, 9 deliberately unresolvable |
| `sample-data-large.csv` | 403 rows, for exercising the free-tier flow cap |

All three carry a `DestThickness` column.

Keep the `.pbix` here rather than in the visual's repository: that one is private, so nothing in
it can be linked to or downloaded by a customer.

## The file is behind the visual

Run `node scripts/verify-pbix.mjs` from the visual's repository to see exactly how far. Today:

```
FAIL  embedded visual is behind - no destinationThickness role (it has 17 of 18)
FAIL  embedded visual has no general, licensing formatting object
warn  never demonstrated: originLocation, destLocation, destinationThickness
```

Nothing about opening the report says any of this. A `.pbix` carries its own private copy of the
visual, frozen at whatever was installed the day it was saved, and **the version number does not
move** - the embedded copy reports 1.0.0.0 exactly as the current build does. The report renders,
the fields bind, and the roles it cannot offer are simply absent from a field well nobody is
looking at. That is how a submission comes to demonstrate a visual that no longer exists.

## Rebuilding it

Power BI Desktop is the only thing that can write this file. The report definition inside it is
editable JSON, but the semantic model is a compiled binary part, so a column cannot be added to it
from outside - which is exactly what a rebuild needs.

1. Open `flow-map-sample.pbix` in Desktop.
2. **Import the current visual.** Format pane -> Get more visuals -> Import a visual from a file,
   and pick the `.pbiviz` from the visual repository's `dist/`. Importing over the existing one
   keeps every field binding and format setting; deleting the visual first would lose them for
   nothing.
3. **Repoint the data.** Transform data -> the `sample-data` query -> Source, and point it at the
   current `sample-data.csv`. `DestThickness` arrives with it. Close & Apply.
4. **Bind Destination Thickness** to `DestThickness` on the Basic Flow Map and Great Circle pages.
   The automatic fan needs no binding at all and appears on its own once the multi-leg rows are in.
5. **Add a place-names page.** Load `sample-data-locations.csv` as a second table and bind
   `OriginLocation` -> Origin Location and `DestLocation` -> Destination Location, with no
   coordinates bound. This is the only way a reviewer sees that the visual resolves names offline,
   and nine of those rows fail on purpose, so the unresolved-rows notice shows up with it.
6. **Rename the banner.** Each page carries a text box reading *Flow Map by Friven*, which is
   also what the four images in `screenshots/` show - they are on the public site and in the
   listing. Retake them once the banner says **FlowPath by Friven**.
7. Save, and run the check again.

The four existing pages and their settings, for reference - they are worth preserving:

| Page | What it shows |
|---|---|
| Basic Flow Map | Coordinates, sizes, legend. Map labels on |
| Animation and Timeline | Animation on, custom water color, legend on top |
| Great Circle - Latitude Change | Great-circle paths, cumulative timeline, centered on -120 |
| Zoom | The same settings, zoomed in |

## Checking it afterwards

```bash
node scripts/verify-pbix.mjs
```

Reads the visual embedded in the report, compares its data roles and formatting objects with
`capabilities.json`, and lists any role the report never binds - a feature the sample cannot show
is one a reviewer has to take on trust. It is deliberately not part of `npm run verify`, because it
needs this repository cloned beside the visual's own.
