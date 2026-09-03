# Sample report

`flow-map-sample.pbix` — the report Microsoft asks for with a Power BI visual submission, and
the one linked from the user guide.

It must work with no network access: import the data rather than linking to a source, so the
file opens and renders for someone who has never seen the CSV.

The data it is built from lives in the visual's own repository:

| File | Purpose |
|---|---|
| `sample-data.csv` | 57 rows of coordinate-based flows, including 3 multi-modal lanes and 4 carrying a transit loss |
| `sample-data-locations.csv` | 48 rows using the `City\|State\|Country` fields, 9 deliberately unresolvable |
| `sample-data-large.csv` | 403 rows, for exercising the flow limit |

All three now carry a `DestThickness` column.

**The `.pbix` is behind the CSVs.** It was built before Destination Thickness, the multi-modal
lanes and the automatic fan existed, so it demonstrates none of them. Rebuilding it means
re-importing `sample-data.csv` and binding `DestThickness` to Destination Thickness — the fan
needs no binding and appears on its own once the new rows are in.

Keep the `.pbix` here rather than in the visual's repository: that one is private, so nothing in
it can be linked to or downloaded by a customer.
