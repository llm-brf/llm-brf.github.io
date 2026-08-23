# Geographic coverage

The service does **not** cover all of Sweden. Coverage is currently limited to
**selected kommuner in the Stockholm region**, and grows over time as new areas
are imported.

![Map of covered listings](./coverage-map.png)

## Currently covered

| Kommun | Scope |
| --- | --- |
| Stockholm | Whole kommun |
| Solna | Whole kommun — except **sold** history in Järvastaden, Bergshamra and Vireberg (see below) |
| Nacka | Whole kommun |
| Tyresö | Whole kommun |

**Known scope exception (v0.4.5).** For-sale advertisements cover all four
kommuner in full. **Solna's *sold* history omits Järvastaden, Bergshamra and
Vireberg** — those districts were missing from the area driver used to build the
sold dataset, so `GET /sales`-derived results and Solna sold statistics exclude
them. Current advertisements in those districts *are* present.

The authoritative, always-current list is the live API itself:

```http
GET /kommuns
```

returns every kommun present in the data (with per-kommun statistics). Treat
that endpoint — not this page — as ground truth when querying.

## Data completeness within covered areas

Coverage of an area does not mean every field is filled. The main historical gap
was the **monthly fee (avgift) on sold apartments**, backfilled progressively per
kommun. As of v0.4.4 all covered kommuns — including the newly whole-kommun
Stockholm — are **essentially complete for recent sales (~97–98% of the last five
years of sold bostadsrätt)**. The deeper pre-2021 tail is intentionally not
backfilled (recent avgift is the analytically useful slice), so fee-derived
statistics are most reliable for the last five years — see the changelog's *Known
limitations* for current status.

## Interpreting empty results

If you query a kommun or area that is not covered, the API returns an empty
result set — **an empty answer means "outside coverage", not "no such
properties exist"**. Before concluding anything from an empty response, check
`GET /kommuns` (and `GET /areas?kommun=<Name>`) to confirm the location is in
the dataset.

## Expansion

More kommuner are imported over time. Additions are announced in the
[changelog](./CHANGELOG.md) and reflected immediately in `GET /kommuns` and on
the map above.
