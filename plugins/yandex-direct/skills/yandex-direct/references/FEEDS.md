# Feeds and product ads

Read this reference when adding or updating a catalog, creating product ads,
changing product selection, or investigating why there are no impressions
after product ads were created. First work out what is being changed:

| Object | What it stores | Command |
|---|---|---|
| Source catalog | Products, prices, availability, images and pages | Edit on the site or in the source file |
| Feed in the account library | Catalog source, business type, processing | `feeds.py` |
| Product ad `ShoppingAd` | Linked feed, filters, text sources and default text | `shopping.py` |
| `ShoppingAd` and `ListingAd` extensions | Sitelinks and callouts | `extensions.py bind` |
| UPC (ЕПК, unified performance campaign) group `UNIFIED_AD_GROUP` | Regions and display conditions | `ads_write.py group` |

In a UPC, `FeedId` and `FeedFilterConditions` belong to the product ad.
Do not write them to `UnifiedAdGroup`. The old `SmartAdGroup` and
`DynamicTextFeedAdGroup` work differently; the rules of those formats do not replace
the UPC mechanics. `shopping.py` works with `ShoppingAd`. `ads.py` reads
catalog `ListingAd` ads; extensions can be bound to them via
`extensions.py bind`, but creating and changing their content still requires the interface.

## Feeds in the library

Commands are run from the skill directory. The examples use placeholder logins,
IDs and URLs; substitute the data of the selected account.

```bash
uv run scripts/feeds.py list --account example --no-cache
uv run scripts/feeds.py get --account example --feed 123 --no-cache --json
uv run scripts/feeds.py add --account example --name 'Каталог товаров' \
  --business-type RETAIL --url https://example.com/catalog.xml  # name: "Product catalog"
uv run scripts/feeds.py add --account example --name 'Каталог из файла' \
  --business-type RETAIL --file catalog.xml  # name: "Catalog from file"
uv run scripts/feeds.py update --account example --feed 123 \
  --url https://example.com/catalog-new.xml --remove-utm-tags YES
uv run scripts/feeds.py delete --account example --feed 123
```

Without `--apply` the change remains a plan; the flag executes a write that was already requested.
Show the contents as "before → after" and the affected campaigns:
updating a shared feed may affect several campaigns. `CampaignIds`
from `Feeds.get` may be incomplete: during testing the field was empty
even for the feed of an active product ad. For a UPC, read `ShoppingAd`
and `ListingAd` via `ads.py` in the campaigns of the selected account and match
`FeedId`; `campaigns.py` returns the list of campaigns. If the check is limited,
state its scope and do not claim that there are no other bindings. Deletion applies
to an unused feed; archiving a campaign does not by itself prove
that the feed is free. `SourceType` and `BusinessType` cannot be changed after creation.
For a different source type, create a new feed. The `RETAIL` value in the example
suits a retail catalog; when copying an existing feed, keep
its actual business type.

The command accepts a public link or a local file. For a source protected by
a login and password, use the Direct interface. `--remove-utm-tags`
is available only for a URL feed. The file is sent whole in base64; the 50 MB limit
applies to the entire request, so the usable size of the source file is smaller.
The plan shows the size and SHA-256. The API does not return the file contents:
checking the name and status is not a byte-by-byte check of the upload.

## Catalog processing

A successful `Feeds.add/update` confirms that the settings were saved, while processing runs
separately. Run
`uv run scripts/feeds.py get --account example --feed 123 --no-cache` again:

| Feed `Status` | Meaning |
|---|---|
| `NEW` | Processing has not started yet |
| `UPDATING` | The catalog is being processed |
| `DONE` | Processing is finished; check the number of products |
| `ERROR` | Processing error; check the source and the message in the account |

Before preparing an ad, get `FilterSchema` and `TitleAndTextSources`:
they define the available selection fields and the sources of the title/text.
Do not invent these fields from the name of the site. If the product count is zero, check
catalog availability, format and contents. Do not promise impressions just because
the feed is saved or processed.

## Product ad

A group allows one non-archived `ShoppingAd`. First read the group
and the existing ads; for a new campaign, follow the usual order of
creation, [choosing placements](PLACEMENTS.md) and [budget management](BUDGETS.md).
The feed and the `ShoppingAd` format do not limit a campaign to the product gallery:
choose placements separately before creating, and for "gallery only" use the explicit
set of settings from the reference. Adding an ad to an existing campaign
does not change its placements.

```bash
uv run scripts/shopping.py get --account example --ad 789 --no-cache --json
uv run scripts/shopping.py add --account example --group 456 --feed 123 \
  --default-text 'Подбор оборудования. Доставка.'  # "Equipment selection. Delivery."
uv run scripts/shopping.py update --account example --ad 789 \
  --default-text 'Подбор оборудования. Доставка и самовывоз.'  # "Equipment selection. Delivery and pickup."
```

Creation requires one default text. It must suit all the
selected products and match the seller's real terms. Sources
are set with the repeatable `--title-source` and `--text-source`; take the valid names
from `TitleAndTextSources` of the processed feed. Fields not passed during an edit
are kept. An explicitly passed list is replaced entirely.

Bind sitelinks and callouts to the ad ID via
`extensions.py bind`. For a new ad, first run `shopping.py add`,
then use the ID you get:

```bash
uv run scripts/extensions.py --account example bind --ad 789 \
  --sitelink-set 102 --callout-id 200 --callout-id 202
```

Creating sets, replacing and removing bindings, and the before/after check are covered in
[EXTENSIONS.md](EXTENSIONS.md). The same command changes the extensions of an existing
`ListingAd`; it keeps the feed, product selection and ad texts.

Saving an ad does not mean impressions have started or moderation has approved it.
Read its actual `State`, `Status` and `FeedProcessingStatus`:
`UNPROCESSED` means processing is not finished yet, `PROCESSED` means it is finished,
`EMPTY_RESULT` means the selection produced no products. To check the content,
show the products and pages that match the filters; check the exact look of the automatically
built cards in the Direct preview.

### Product ad generation and first impressions

Warn the user at creation: **generating product ads
can take up to three days**. This period was stated by Yandex Direct support
in a reply provided by the user on 20.09.2026. It is a guide for the wait,
not a guarantee that impressions will start by a particular hour.

Feed processing, ad generation, moderation and statistics accumulation are
separate stages. The `DONE` status of the feed and the `PROCESSED` status of the ad
do not by themselves promise immediate impressions. If a new product ad
has no impressions, take into account the time since creation and check the actual statuses,
processing errors, filter results, approval for serving and campaign settings.
Fix any error you find immediately; do not blame it on waiting for generation.

If there are no obvious obstacles and the ad was created recently, explain the possible
delay and offer to recheck the statuses and statistics after generation.
If support has already confirmed that generation recently finished, wait for
statistics to accumulate: there is no need to count another three days for generation.
Do not raise the budget or bids just because there are no first impressions:
that needs a separate reason. If impressions have not appeared after three days,
continue diagnosing and, if needed, suggest contacting support
with the creation time and the current statuses.

## Product filters

`--filters filters.json` accepts a JSON array of rules. For example, select two
categories and products priced above a given amount:

```json
[
  {"Operand": "categoryId", "Operator": "EQUALS_ANY", "Arguments": ["10", "20"]},
  {"Operand": "price", "Operator": "GREATER_THAN", "Arguments": ["1000"]}
]
```

This is an example of fields for a product catalog; check them against `FilterSchema`.
Rules are combined with AND; `EQUALS_ANY` allows any of the listed
values. Arguments are written as strings, including numbers. Up to 30 rules and
65 KB of JSON; the valid operators and number of arguments depend on the feed field.
For a range, use `IN_RANGE`, for example `Arguments: ["1000-5000"]`.

```bash
uv run scripts/shopping.py update --account example --ad 789 --filters filters.json
uv run scripts/shopping.py update --account example --ad 789 --clear-filters
```

The file replaces the entire selection. To add a condition, first read the current
rules and include the ones being kept in the final file. Removing filters opens
the selection to the whole feed: explain this expansion to the user explicitly. Before the change,
show the rules before and after; afterwards, reread them and check the processing
result. If the conditions gave an empty selection, do not consider the task done
based only on a successful API response.

In the API, for `Ads.add` filters and text sources are passed as arrays; for
`update/get` they are an object `{"Items": [...]}` or `null`. The command performs
this conversion. Omitting a field in an update means "keep",
`null` means clear. `DefaultTexts` remains an array and may be omitted when
partially editing a `ShoppingAd`.

## Replacing a feed

`FeedId` is absent from `Ads.update`. To replace it, prepare a new ad
with the needed feed, keeping the suitable settings. Account for the limit of one
non-archived product ad per group: agree in advance on replacing within the same
group or creating a separate group. Show the old and new ad,
the filters and the text sources. Archiving the old one and launching the new one are separate
changes; do not stop running ads just to prepare a copy.

Reference: [Feeds](https://yandex.ru/dev/direct/doc/ru/feeds/feeds),
[feed requirements](https://yandex.ru/dev/direct/doc/ru/objects/feed),
[Ads.add](https://yandex.ru/dev/direct/doc/ru/ads/add),
[Ads.update](https://yandex.ru/dev/direct/doc/ru/ads/update),
[selection rules](https://yandex.ru/dev/direct/doc/ru/objects/ad).
