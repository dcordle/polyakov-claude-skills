---
name: yandex-metrika
description: |
  Yandex Metrica: traffic breakdown by sources and UTM tags,
  reports on conversions and search engines, API segments and direct access to counters.
  Cache-first approach to keep the context window clean.
  Triggers: yandex metrica (яндекс метрика), yandex metrika, metrika analytics,
  metrica traffic (метрика трафик), metrica conversions (метрика конверсии), metrica report (метрика отчёт), create a segment (создать сегмент),
  counter access (доступ к счётчику), grant access to an ad account (выдать доступ рекламному кабинету).
---

# yandex-metrika

Working with the Yandex Metrica API: traffic breakdown by sources and UTM tags,
reports on conversions and search engines, API segments and direct access to
counters.

## Config

Requires `YANDEX_METRIKA_TOKEN` in `config/.env`. Reading needs the
`metrika:read` scope; creating segments and managing access needs `metrika:write`.
Instructions: `config/README.md`.

## Philosophy

1. **Cache-first**: configuration data (counters, goals, info) is cached for a long time. Reports are cached by the key counter+dates+params. Always check the cache before an API request.
2. **Context window hygiene**: stdout is limited to 30 lines. Full data goes to CSV/file. The cache can be searched with grep/rg without loading it into the context.
3. **Accurate data**: accuracy=1 (no sampling), the isRobot filter is on by default.
4. **Attribution**: the default is `lastsign` (last significant source). Ask the user on the first run.

## Workflow

### STOP! Before any analysis:

1. **Get the list of counters:**
   ```bash
   bash scripts/counters.sh
   ```

2. **Ask the user** (if the counter is not obvious from the context):
   ```
   "Which counter/site do you mean?
   Give the ID, name or domain."
   ```
   If the user named a site/domain, search with `--search`:
   ```bash
   bash scripts/counters.sh --search "metallik"
   ```
   This is a grep over the TSV (id + name + site), so it also matches by domain.

3. **Get the counter info and its goals:**
   ```bash
   bash scripts/counter_info.sh --counter <ID>
   bash scripts/goals.sh --counter <ID>
   ```

4. **Ask which goals are conversion goals:**
   ```
   "Which of these goals are conversion goals for your business?
   [list of goals from goals.sh]
   I will save the selected ones for future reports."
   ```

5. **Save the configuration** to `cache/counter_<id>/config.json`:
   ```json
   {
     "attribution": "lastsign",
     "conversion_goals": [
       {"id": 12345, "name": "Order placed"},
       {"id": 67890, "name": "Request submitted"}
     ]
   }
   ```

6. **Run the reports** for the user's task.

## Segments and access

For an API segment, first fix a clear name and the `filters` expression.
Creating and deleting without `--apply` shows the plan and changes nothing.
Before writing, the script reads the fresh role on the counter; the API server
remains the final source of permissions.

```bash
bash scripts/segments.sh --counter <ID> --action create \
  --name "Product card visitors" \
  --expression "EXISTS(ym:pv:URL=@'/catalog/')"
# after checking the plan, repeat with --apply
```

Get the exact login from the calling task, then check direct access to the counter
and, if needed, prepare the minimal role:

```bash
bash scripts/grants.sh --counter <ID> --action get --login <LOGIN>
bash scripts/grants.sh --counter <ID> --action add \
  --login <LOGIN> --permission view
# after checking the plan, repeat with --apply
```

`view` is usually enough to use a segment. Do not grant `edit` unless the
task requires changing the counter. If the user chose `analyst` or `edit`,
pass that role through `--permission`; change an existing role with `update`.
The `/grants` list contains direct permissions;
the owner is shown separately, and account representatives are not in this list.
After writing, return to the calling task the counter number, segment ID, login and
the confirmed role.
Details: [API segments](references/SEGMENTS.md) and
[direct access to a counter](references/ACCESS.md).

## Scripts

General call pattern:
```bash
bash scripts/<script>.sh --counter <ID> --date1 YYYY-MM-DD [--date2 ...] [--group month] [--csv path]
```

| Script | Description | Special params |
|--------|-------------|----------------|
| `counters.sh` | List of counters | `--search "query"` |
| `goals.sh` | Counter goals | - |
| `counter_info.sh` | Counter metadata | - |
| `traffic_summary.sh` | Traffic by sources | - |
| `conversions.sh` | Goal completions | `--goals "ID,ID"` / `--all-goals`; defaults to `config.json` |
| `utm_report.sh` | UTM breakdown | - |
| `search_engines.sh` | Search engines (organic) | - |
| `ecommerce.sh` | Purchases, revenue, average order value | `--currency RUB\|USD\|EUR`; auto from counter_info |
| `direct_clients.sh` | Direct logins | - |
| `direct_costs.sh` | Direct costs (`ym:ad:*`) | `--direct-client-logins "login"`; no `--group`/`--device`/`--source` |
| `comparison.sh` | Comparison of two periods | `--date1a/--date2a/--date1b/--date2b`; `--dimension`, `--metrics` |
| `segments.sh` | List, view, create and delete API segments | `--action`; writes only with `--apply` |
| `grants.sh` | Owner and direct access by login; roles `view`, `analyst`, `edit` | `--action`; writes only with `--apply` |

Not all scripts support all the general parameters: see **Special params**.
For reliable JSON parsing, the management scripts `segments.sh` and `grants.sh`
need Python 3.10+ or `uv`; regular reports still work without them.

### Goals report

`conversions.sh` automatically requests goals in batches of six and merges them
into one CSV. This works for `--all-goals`, `--goals` and goals from `config.json`.
With `--all-goals`, the goal list is refreshed through the Management API first.
The merge is done with plain `awk`; Python and `uv` are not needed to run reports.
Three metrics are saved for each goal: visits with a completion, completions and conversion.
The order of goals in the columns matches the selected list.

A total visits metric is added to the requests so that sources with zero
completions do not disappear from individual batches. The final CSV has no service metrics,
but sources without conversions are kept; the overall conversion includes their visits.
Sources are sorted by visits with a completion of the first selected goal,
and by source when they are equal.

`--limit` caps the number of sources: without `--group` the allowed range is 1-100000
(default 100); with `--group` it is 1-30 (default 30, the API parameter `top_keys`).
The dates are not shortened. The report goes to the cache and `--csv` only after
all batches have been fetched and merged successfully.

## General parameters of report scripts

| Param | Required | Default | Values |
|-------|----------|---------|--------|
| `--counter` | yes | - | counter ID |
| `--date1` | yes | - | YYYY-MM-DD |
| `--date2` | no | today | YYYY-MM-DD |
| `--group` | no | - | day, week, month |
| `--device` | no | all | desktop, mobile, tablet |
| `--source` | no | all | organic, ad, referral, direct, social |
| `--attribution` | no | lastsign | lastsign, last, first |
| `--filters` | no | no robots | report API segmentation expression |
| `--limit` | no | API default | number of rows |
| `--csv` | no | - | export path |
| `--no-cache` | no | - | skip the cache |

## Cache strategy

The cache is stored in `cache/`:
- `counters.json` + `counters.tsv`: all counters
- `counter_<id>/info.json`: metadata (permanent)
- `counter_<id>/goals.json` + `goals.tsv`: goals
- `counter_<id>/config.json`: attribution, conversion goals
- `counter_<id>/direct_clients.json`: Direct logins
- `counter_<id>/reports/*.csv`: report results

Segments and access are always read fresh and are not cached: this data
changes independently of reports, and the access list contains user logins.

To search the cache: `grep "text" cache/counters.tsv` or `rg "text" cache/`.

## Advanced scenarios

- [Popular search queries](references/SEARCH_QUERIES.md)
- [Custom reports and JSON requests](references/CUSTOM_REPORTS.md) (drilldown, metrika_get, etc.)
- [Dimensions/metrics reference](references/API_REFERENCE.md)
- [Year-over-year period comparison](references/PERIOD_COMPARISON.md)
- [Direct costs and PnL](references/DIRECT_COSTS.md)
- [API segments for audience selection in Direct](references/SEGMENTS.md)
- [Checking and granting access to ad accounts](references/ACCESS.md)
- [API limitations](references/API_REFERENCE.md#known-api-limitations) (bytime, scope mixing, drilldown CSV)

## API limits

- **Reporting API**: ~200 requests / 5 minutes (if exceeded, wait ~5 minutes)
- The scripts handle 429 automatically (Retry-After ≤ 60s: retry, otherwise fail with a message)
