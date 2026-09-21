# Sample report

`flowpath-sample.pbix` — the report Microsoft asks for with a Power BI visual submission, and
the one linked from the user guide.

It must work with no network access: import the data rather than linking to a source, so the
file opens and renders for someone who has never seen the CSV.

The data it is built from lives in the visual's own repository:

| File | Purpose |
|---|---|
| `sample-data.csv` | 66 rows: 61 flows with **both** coordinates and `City\|State\|Country` names, 4 multi-leg routes, 9 carrying a transit loss, and 5 rows that cannot be placed on purpose. The one table the report needs |
| `sample-data-locations.csv` | 48 rows using place names only, 9 deliberately unresolvable. For the geocoding test plan, not the report |
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

1. Open `flowpath-sample.pbix` in Desktop.
2. **Import the current visual.** Format pane -> Get more visuals -> Import a visual from a file,
   and pick the `.pbiviz` from the visual repository's `dist/`. Importing over the existing one
   keeps every field binding and format setting; deleting the visual first would lose them for
   nothing.
3. **Repoint the data.** Transform data -> the `sample-data` query -> Source, and point it at the
   current `sample-data.csv`. `DestThickness`, `OriginLocation` and `DestLocation` arrive with it.
   Close & Apply.
4. **Bind Destination Thickness** to `DestThickness` on the Basic Flow Map and Great Circle pages.
   The automatic fan needs no binding at all and appears on its own once the multi-leg rows are in.
5. **Add a place-names page** from the same table: bind `OriginLocation` -> Origin Location and
   `DestLocation` -> Destination Location, and **no coordinates** - coordinates win wherever they are
   bound, and the page exists to show names resolving without them. It draws the same map as the
   coordinates page, plus a notice: five rows are unplaceable on purpose, and hovering the notice
   names each one and why. No second table, no relationship.
6. **Put something on the page for the map to filter.** A table beside the map on at least one
   page - Origin, Destination and the thickness measure is enough. See below: this is what the
   first certification pass failed on.
7. **Add the hints and tips text box.** Copy below. Certification asks for it by name.
8. Save, and run the check again.

The four existing pages and their settings, for reference - they are worth preserving:

| Page | What it shows |
|---|---|
| Basic Flow Map | Coordinates, sizes, legend. Map labels on |
| Animation and Timeline | Animation on, custom water color, legend on top |
| Great Circle - Latitude Change | Great-circle paths, cumulative timeline, centered on -120 |
| Zoom | The same settings, zoomed in |

## What certification asked for

The first submission passed with two soft failures, both about this file rather than the visual:

> **1180.2.2.3 Core Functions - Filter Out.** Your visual does not appear to filter outwards to
> other visuals.

True of the report, not of the code. Every page held the map and nothing else, so a reviewer
clicking an arrow saw nothing change. The visual calls `selectionManager.select()` on both arrows
and markers and declares `supportsMultiVisualSelection`; it has nothing to filter here. **One table
on one page fixes it** - though putting one on every page is better, since a reviewer may open any
of them.

> **1180.2.3.1 Sample File Hints and Tips.** We recommend including hints and tips on how to use
> the visual within your sample file.

A text box carries them. Paste this on the first page:

```
How to use FlowPath

- Click an arrow or a bubble to cross-filter the rest of the page. Ctrl+click adds to the
  selection; click empty space to clear it.
- Hover any arrow for origin, destination, volume and category.
- Bind Latitude and Longitude at each end, or a single City|State|Country column - place names
  resolve inside the visual, with no geocoding service called.
- Route ID and Leg Order chain legs into one multi-hop route rather than three separate arrows.
- Start Time drives the timeline. Press play, or drag the scrubber to filter the whole page.
- Rows that cannot be located are counted at the bottom left - hover for the reason.
```

**The text box alone did not satisfy it.** 1180.2.3.1 came back a second time against a report
that had one on the first page. Microsoft's own checklist for the sample file says what they look
for: *"a 'hints' page at the end with some tips and tricks and things to avoid."* A page, named for
it, last - and with a "things to avoid" section, which is the part a how-to text box does not have.

Add a fifth page called **Hints**, after Great Circle, holding one text box:

```
Hints

TIPS
- Bind Latitude and Longitude at each end, or one City|State|Country column. Where both are bound, coordinates win.
- Click an arrow or a location to filter the rest of the page. Ctrl+click adds to the selection; click empty space to clear it.
- Hover any arrow for origin, destination, volume and category.
- Route ID and Leg Order chain legs into one multi-hop route. Start Time turns on the timeline.
- Destination Thickness draws a lane between its two widths, colored by what it gained or lost in transit.

TRICKS
- Click a legend entry to select every flow in that category at once.
- Turn on "Split inbound and outbound on select" on the Arrows card, then click a hub: inbound traffic redraws as a dashed centerline, outbound keeps its body.
- Switch Path mode to Great circle for lanes that cross the dateline. They stop doubling back across the whole map.
- Set Center longitude on the Basemap card to put your region in the middle instead of the Atlantic.
- Right-click an arrow for Include or Exclude. That filters this visual only, not the page.

THINGS TO AVOID
- A date hierarchy in Start Time. It arrives as four separate columns and the timeline cannot use it. Use the dropdown on the field well to pick the date field itself.
- A place name without enough to pin it down. "Springfield" alone is refused rather than guessed; "Springfield|IL|USA" resolves. The notice at the bottom left names each row that could not be placed.
- More than 10,000 rows. Power BI stops sending at that point and the map says "Showing part of the data". Aggregate in the model first.
- Summing a measure that should not be summed. Bubble sizes add up across every flow at a location, so set a field like population to Max in the field well.
- Expecting a location column to override coordinates. It never does. Clear the coordinate fields to use names.

Watermarked features are licensed. They render in full before purchase so you can evaluate them.
```

`verify-pbix.mjs` checks for all of it - a second visual, a text box, and a hints page last - so
none can be lost in a later rebuild.

## Checking it afterwards

```bash
node scripts/verify-pbix.mjs
```

Reads the visual embedded in the report, compares its data roles and formatting objects with
`capabilities.json`, and lists any role the report never binds - a feature the sample cannot show
is one a reviewer has to take on trust. It is deliberately not part of `npm run verify`, because it
needs this repository cloned beside the visual's own.
