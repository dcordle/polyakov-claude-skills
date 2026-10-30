# Direct API services

For daily work use the ready-made commands from [SKILL.md](../SKILL.md). This map shows what the official API can do, including operations that have no ready-made command in the skill. The full map of methods, URLs and call costs: [api_map.json](api_map.json). The action availability map: [coverage.json](coverage.json).

## URLs and versions

Main JSON API: `https://api.direct.yandex.com/json/v501/{service}`. Reports: `https://api.direct.yandex.com/json/v5/reports`. Use the documented URL of the service you need. The fact that another URL responds does not guarantee the same behavior.

A regular request contains `method` and `params`; successful data is in `result`, a general error is in `error`. For a batch change, also check `Errors` and `Warnings` of each object. The headers for choosing the account are described in [CLIENT_LOGIN.md](CLIENT_LOGIN.md), points and retries in [ERRORS_AND_LIMITS.md](ERRORS_AND_LIMITS.md).

## v501 services

| Service | Purpose | Methods |
|---|---|---|
| [AdExtensions](https://yandex.ru/dev/direct/doc/ru/adextensions/adextensions) | Working with ad extensions. Currently there is one extension type: callout. | `add`, `delete`, `get` |
| [AdGroups](https://yandex.ru/dev/direct/doc/ru/adgroups/adgroups) | Working with ad groups. | `add`, `delete`, `get`, `update` |
| [AdImages](https://yandex.ru/dev/direct/doc/ru/adimages/adimages) | Working with images. | `add`, `delete`, `get` |
| [Ads](https://yandex.ru/dev/direct/doc/ru/ads/ads) | Working with ads. | `add`, `archive`, `delete`, `get`, `moderate`, `resume`, `suspend`, `unarchive`, `update` |
| [AdVideos](https://yandex.ru/dev/direct/doc/ru/advideos/advideos) | Working with videos. | `add`, `get` |
| [AgencyClients](https://yandex.ru/dev/direct/doc/ru/agencyclients/agencyclients) | The service is meant for agencies and lets them manage agency clients. | `add` (removed), `addPassportOrganization`, `addPassportOrganizationMember`, `get`, `update` |
| [AudienceTargets](https://yandex.ru/dev/direct/doc/ru/audiencetargets/audiencetargets) | Working with audience targeting conditions. | `add`, `delete`, `get`, `resume`, `setBids`, `suspend` |
| [Bids](https://yandex.ru/dev/direct/doc/ru/bids/bids) | The service is meant for setting bids on keywords and autotargeting. | `get`, `set`, `setAuto` |
| [Businesses](https://yandex.ru/dev/direct/doc/ru/businesses/businesses) | The service returns business organization profiles. | `get` |
| [BidModifiers](https://yandex.ru/dev/direct/doc/ru/bidmodifiers/bidmodifiers) | The service is meant for managing bid adjustments. | `add`, `delete`, `get`, `set` |
| [Campaigns](https://yandex.ru/dev/direct/doc/ru/campaigns/campaigns) | Working with campaigns. | `add`, `archive`, `delete`, `get`, `resume`, `suspend`, `unarchive`, `update` |
| [Changes](https://yandex.ru/dev/direct/doc/ru/changes/changes) | The service is meant for checking whether anything has changed. | `checkDictionaries`, `checkCampaigns`, `check` |
| [Clients](https://yandex.ru/dev/direct/doc/ru/clients/clients) | The service is meant for managing advertiser parameters and the settings of a user who represents the advertiser, and for getting agency parameters and the settings of a user who represents the agency. | `get`, `update` |
| [Creatives](https://yandex.ru/dev/direct/doc/ru/creatives/creatives) | Working with creatives. | `add`, `get` |
| [Dictionaries](https://yandex.ru/dev/direct/doc/ru/dictionaries/dictionaries) | The service is meant for getting reference data: regions, time zones, exchange rates, metro stations, limits on parameter values, etc. | `get`, `getGeoRegions` |
| [Feeds](https://yandex.ru/dev/direct/doc/ru/feeds/feeds) | Working with feeds. | `add`, `delete`, `get`, `update` |
| [KeywordBids](https://yandex.ru/dev/direct/doc/ru/keywordbids/keywordbids) | The service is meant for setting bids on keywords and autotargeting and for getting data useful when setting bids. | `get`, `set`, `setAuto` |
| [Keywords](https://yandex.ru/dev/direct/doc/ru/keywords/keywords) | Working with keywords and autotargeting. | `add`, `delete`, `get`, `resume`, `suspend`, `update` |
| [KeywordsResearch](https://yandex.ru/dev/direct/doc/ru/keywordsresearch/keywordsresearch) | The service is meant for preprocessing and researching keywords. | `deduplicate`, `hasSearchVolume` |
| [Leads](https://yandex.ru/dev/direct/doc/ru/leads/leads) | The service returns data that users entered in forms on Turbo pages. | `get` |
| [NegativeKeywordSharedSets](https://yandex.ru/dev/direct/doc/ru/negativekeywordsharedsets/negativekeywordsharedsets) | Working with negative keyword sets. | `add`, `delete`, `get`, `update` |
| [RetargetingLists](https://yandex.ru/dev/direct/doc/ru/retargetinglists/retargetinglists) | Working with retargeting and audience selection conditions. | `add`, `delete`, `get`, `update` |
| [Sitelinks](https://yandex.ru/dev/direct/doc/ru/sitelinks/sitelinks) | Working with sitelink sets. | `add`, `delete`, `get` |
| [Strategies](https://yandex.ru/dev/direct/doc/ru/strategies/strategies) | Working with portfolio strategies. | `add`, `archive`, `get`, `unarchive`, `update` |
| [TurboPages](https://yandex.ru/dev/direct/doc/ru/turbopages/turbopages) | The service returns Turbo page parameters. | `get` |

## Legacy ad types

`DynamicTextAdTargets`, `SmartAdTargets` and `VCards` still have separate mentions and schemas, but full reference pages exist not for all operations. The presence of a WSDL alone does not tell you what the fields mean or what limits apply. To make changes, use only a supported command with a verifiable result; otherwise show the path in the interface.

`ResponsiveAd` is the main format for writing ads. Text-and-image ads can be converted into it automatically; updating through the old `TextAd` structure risks shrinking the set of headlines, texts and assets.

## Reports

Reports is a separate service, not a JSON `get` method. It returns TSV and can prepare a report in a queue. Goals, attribution models, fields, periods and waiting are described in [REPORTS.md](REPORTS.md). Do not treat the cost in points as zero just because a row is missing from the cost table; respect the rate limits of Reports itself.

## Remaining v4 methods

Live v4 calls go to `https://api.direct.yandex.com/live/v4/json/`; some methods use `/v4/json/`. The legacy management of campaigns, ads and bids does not replace v501.

Version 4 differs in shape: `token`, `method`, `param`, `locale` are passed in the body, successful data is in `data`, errors are in the root `error_code`, `error_str`, `error_detail`. The standard client hides the token; do not assemble such a request by hand in a command you print.

| Area | Reference methods |
|---|---|
| Shared account | `EnableSharedAccount`, `AccountManagement` |
| Financial operations | `CreateInvoice`, `PayCampaigns`, `TransferMoney`, `GetCreditLimits` |
| Budget forecast | `CreateNewForecast`, `GetForecast`, `GetForecastList`, `DeleteForecastReport` |
| Keyword suggestions and query statistics | `GetKeywordsSuggestion`, `CreateNewWordstatReport`, `GetWordstatReport`, `GetWordstatReportList`, `DeleteWordstatReport` |
| Ad and campaign tags | `GetBannersTags`, `UpdateBannersTags`, `GetCampaignsTags`, `UpdateCampaignsTags` |
| Retargeting | `GetRetargetingGoals` |
| Points | `GetClientsUnits` |
| Dictionaries | `GetRegions`, `GetStatGoals`, `GetRubrics`, `GetTimeZones` |
| API information | `GetAvailableVersions`, `GetVersion`, `PingAPI`, `GetEventsLog` |

`GetRetargetingGoals` is used for available goals when this source is needed. v4 points form a separate pool: `GetClientsUnits` does not show the v5 balance. The v5 balance is read from the `Units` header, the daily limit from `Clients.get → Restrictions → API_POINTS`.

For images use `AdImages` and `Ads` v501. The old v4 `AdImage` is not considered a working replacement. Take regions and time zones through `Dictionaries` v501.

Financial methods are listed to show the boundaries of the API; their presence in the reference is not permission to pay or transfer funds. This skill does not perform such actions.

Sources: [v5 reference](https://yandex.ru/dev/direct/doc/ru/), [v4 methods](https://yandex.ru/dev/direct/doc/dg-v4/ru/reference/_AllMethods).
