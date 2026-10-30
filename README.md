# polyakov-claude-skills

A set of skills for Claude Code.

## Contents

- [Installation](#installation)
  - [Via the marketplace (recommended)](#via-the-marketplace-recommended)
  - [Manual install (without the marketplace)](#manual-install-without-the-marketplace)
  - [Local testing](#local-testing)
- [Available skills](#available-skills)
  - [blogger-ads-skill](#blogger-ads-skill) - support for advertising placements
  - [docx-contracts](#docx-contracts) - filling Word templates
  - [scrapedo-web-scraper](#scrapedo-web-scraper) - web scraping through Scrape.do
  - [agent-deck](#agent-deck) - managing AI agent sessions
  - [genome-analizer](#genome-analizer) - genetic data analysis
  - [ssh-remote-connection](#ssh-remote-connection) - SSH connection to servers
  - [yandex-wordstat](#yandex-wordstat) - search demand analysis
  - [codex-review](#codex-review) - cross-agent review
  - [fal-ai-image](#fal-ai-image) - image generation
  - [yandex-search-api](#yandex-search-api) - parsing Yandex search results
  - [yandex-direct](#yandex-direct) - statistics and management for Yandex Direct
  - [yandex-metrika](#yandex-metrika) - Yandex Metrica reports, segments and access
  - [yandex-webmaster](#yandex-webmaster) - managing sites in Yandex Webmaster
  - [zoomkit](#zoomkit) - balance, invoices, reports and Yandex Direct settings through the ZoomKit API
  - [telegraph-publisher](#telegraph-publisher) - publishing to Telegraph
  - [crawl4ai-seo](#crawl4ai-seo) - SEO site crawler
  - [telegram-channel-parser](#telegram-channel-parser) - parsing Telegram channels
  - [x-research](#x-research) - X/Twitter research through the xAI Grok API
  - [github-pages-publisher](#github-pages-publisher) - publishing to GitHub Pages
  - [sourcecraft-publisher](#sourcecraft-publisher) - publishing to SourceCraft Sites
  - [reddit-skill](#reddit-skill) - Reddit without keys: posts, comments, RSS; OAuth2 API
  - [knowledge-compiler](#knowledge-compiler) - compiling books and long sources into personal skills
  - [perplexity-search](#perplexity-search) - search and research through the Perplexity API
- [Repository structure](#repository-structure)
- [License](#license)

## Installation

### Via the marketplace (recommended)

```bash
# Add the marketplace
/plugin marketplace add artwist-polyakov/polyakov-claude-skills

# Install the plugins you need
/plugin install blogger-ads-skill
/plugin install docx-contracts
/plugin install scrapedo-web-scraper
/plugin install agent-deck
/plugin install genome-analizer
/plugin install ssh-remote-connection
/plugin install yandex-wordstat
/plugin install yandex-search-api
/plugin install yandex-direct
/plugin install yandex-metrika
/plugin install codex-review
/plugin install fal-ai-image
/plugin install yandex-webmaster
/plugin install zoomkit
/plugin install telegraph-publisher
/plugin install crawl4ai-seo
/plugin install telegram-channel-parser
/plugin install x-research
/plugin install github-pages-publisher
/plugin install sourcecraft-publisher
/plugin install reddit-skill
/plugin install knowledge-compiler
/plugin install perplexity-search
```

### Manual install (without the marketplace)

If you don't want to use the marketplace, copy the skill folder into the `.claude/skills/` directory:

**Globally (for all projects):**
```bash
# Create the directory if it does not exist
mkdir -p ~/.claude/skills

# Copy the skill you need
cp -r plugins/agent-deck/skills/agent-deck ~/.claude/skills/
```

**For a specific project:**
```bash
# In the project root
mkdir -p .claude/skills

# Copy the skill
cp -r plugins/genome-analizer/skills/genome-analizer .claude/skills/
```

After copying, Claude Code picks the skill up automatically on the next start.

### Local testing

```bash
claude --plugin-dir ./plugins/agent-deck
```

---

## Available skills

### [blogger-ads-skill](plugins/blogger-ads-skill/skills/blogger-ads-skill)

Support for an advertising deal in Telegram on the side of the blogger, the advertiser or the agency.

- Separate DOCX templates for the contract and two appendices, with columns and fonts preserved
- Map of insertion points and questions about terms, VAT, deductions, labeling and document exchange
- Checking the received version against quotes and pages, with a single source for the table of comments and ready-made edits
- Invoice and printable UPD (universal transfer document), UPD seller-title XML validated against the official XSD
- Deal state, confirmation checks, deadline recalculation and a reminders file for the calendar
- Handoff check: verbatim instructions, checksums, current document versions and upcoming actions

Scripts run through `uv` and do not depend on `docx-contracts`. Before a real placement the agent checks the current rules and the agreed terms. The XML must be signed and its acceptance verified by the EDO (electronic document exchange) operator.

**Examples:** "подготовь договор на рекламу" ("prepare an advertising contract"), "сделай счёт и УПД" ("make an invoice and UPD"), "перенеси размещение и напоминания" ("move the placement and reminders"), "проверь, можно ли публиковать" ("check whether it can be published").

---

### [docx-contracts](plugins/docx-contracts/skills/docx-contracts)

Filling Word templates (contracts, forms) from data in the context.

- Substitutes values into `{{VARIABLE}}` placeholders
- Extracts the schema from the template
- Asks for missing data

**Triggers:** uploading a .docx file with placeholders

---

### [scrapedo-web-scraper](plugins/scrapedo-web-scraper/skills/scrapedo-web-scraper)

Web scraping through Scrape.do with bot-protection bypass and JavaScript rendering.

- Bypasses blocks and CAPTCHAs
- Supports JavaScript rendering
- Extracts text from HTML

**Triggers:** when a plain fetch does not work

---

### [agent-deck](plugins/agent-deck/skills/agent-deck)

Managing AI agent sessions through the agent-deck CLI.

- Creating and launching child Claude sessions
- Tracking status and collecting results
- Connecting MCP servers
- Parent-child session hierarchy

**Triggers (RU):**
- "запусти агента" / "запусти саб-агента" ("launch an agent" / "launch a sub-agent")
- "проверь сессию" / "проверь статус" ("check the session" / "check the status")
- "покажи вывод агента" ("show the agent's output")

**Triggers (EN):**
- "launch sub-agent" / "create sub-agent"
- "check session" / "show agent output"

---

### [genome-analizer](plugins/genome-analizer/skills/genome-analizer)

Analysis of genetic data from a VCF file.

- SNP search by the topic of the question (GWAS Catalog, SNPedia)
- Genotype interpretation
- Generating personalized reports with recommendations

**Triggers (RU):**
- "проанализируй мой геном" ("analyze my genome")
- "что у меня с генетикой по [теме]" ("what does my genetics say about [topic]")
- "мой генотип для [признака]" ("my genotype for [trait]")

**Triggers (EN):**
- "analyze my genome"
- "what's my genetics for [topic]"

---

### [ssh-remote-connection](plugins/ssh-remote-connection/skills/ssh-remote-connection)

SSH connection to remote servers by key or password.

- Running commands on the remote server
- Agent forwarding (`-A`) to use local SSH keys
- Login/password connection through `SSH_PASSWORD` when `sshpass` or `expect` is available
- Docker container management, log viewing

**Triggers (RU):**
- "выполни на сервере" ("run on the server")
- "проверь логи на сервере" ("check the logs on the server")
- "перезапусти сервис" ("restart the service")

**Triggers (EN):**
- "run on server"
- "check server logs"
- "restart service"

---

### [yandex-wordstat](plugins/yandex-wordstat/skills/yandex-wordstat)

Search demand analysis through the Wordstat API in the Yandex Cloud Search API. [Access setup](plugins/yandex-wordstat/skills/yandex-wordstat/config/README.md) uses a service account key.

- Top search queries for a phrase
- Demand dynamics by month
- Regional statistics
- Intent check through web search

**Triggers (RU):**
- "проанализируй спрос на" ("analyze demand for")
- "найди запросы для рекламы" ("find queries for ads")
- "какой спрос на [тему]" ("what is the demand for [topic]")

**Triggers (EN):**
- "analyze search demand"
- "find keywords for"

---

### [codex-review](plugins/codex-review/skills/codex-review)

Cross-agent review: Claude implements, Codex (GPT-5.2) reviews.

- Workflow: init session → plan review → implementation → code review
- Local review log in `.codex-review/<branch>/notes/`
- Anti-recursion through the `CODEX_REVIEWER` env guard

**Triggers (RU):**
- "кодекс ревью" ("codex review")

**Triggers (EN):**
- "with codex review"
- "codex review workflow"
- "start codex review"

---

### [fal-ai-image](plugins/fal-ai-image/skills/fal-ai-image)

Image generation through fal.ai with the model switchable from the config.

- Google Nano Banana Pro by default, for backward compatibility
- OpenAI GPT Image 2 through the same `FAL_KEY`
- Text-to-image generation and editing from references
- Old `--aspect-ratio` / `--resolution` workflows keep working after switching to GPT
- One-off model override with `--model gpt|openai|nano-banana|google|gemini`

**Triggers (RU):**
- "сгенерируй изображение" ("generate an image")
- "нарисуй картинку" ("draw a picture")
- "создай инфографику" ("create an infographic")

**Triggers (EN):**
- "generate image"
- "create infographic"
- "draw a picture"

---

### [yandex-search-api](plugins/yandex-search-api/skills/yandex-search-api)

Parsing Yandex search results through the Yandex Cloud Search API v2.

- Synchronous and asynchronous search modes
- Authorization through an IAM token (JWT PS256 from a Service Account Key)
- SERP parsing: position, title, URL, snippet
- Result caching and resumable async

**Triggers (RU):**
- "поиск в яндексе" ("search in Yandex")
- "выдача яндекса по запросу" ("Yandex results for a query")
- "парсинг выдачи" ("parse search results")

**Triggers (EN):**
- "yandex search api"
- "parse yandex serp"

---

### [yandex-direct](plugins/yandex-direct/skills/yandex-direct)

Statistics and ad management through the official Yandex Direct API.
The instructions explain the general order of analysis, the metric formulas and how to work
with ad objects. Conversions are counted by the goals the user selects.
Available: audit, period comparison, analysis of demand and positions, creating and changing
campaigns, ads, keywords and bids, with a "before/after" view.

Also kept: campaign export, negative keywords and cross-negation, working with several
accounts, preparing ad sets and HTML preview. Full data is
stored locally; the chat gets a short summary. Requires Python 3.11+ or `uv`,
no third-party libraries.

[Instructions](plugins/yandex-direct/skills/yandex-direct/SKILL.md) ·
[Connection](plugins/yandex-direct/skills/yandex-direct/config/README.md)

---

### [yandex-metrika](plugins/yandex-metrika/skills/yandex-metrika)

Yandex Metrica analytics and management: reports, API segments and access.

- Cache-first strategy with TSV indexes for grep
- Reports: traffic by source, conversions by goal, UTM tagging, search engines
- Filters: device, source, attribution model, robots excluded by default
- The cache is skipped automatically for the current date
- Create segments from `filters` expressions to build audiences in Direct
- Check logins, grant and change direct access with a choice of role

**Triggers (RU):**
- "покажи трафик по счётчику" ("show traffic for the counter")
- "конверсии за период" ("conversions for a period")
- "аналитика метрики" ("Metrica analytics")
- "создай сегмент для Директа" ("create a segment for Direct")
- "проверь и выдай доступ к счётчику" ("check and grant access to the counter")

**Triggers (EN):**
- "yandex metrika analytics"
- "show traffic sources"
- "conversion report"

---

### [yandex-webmaster](plugins/yandex-webmaster/skills/yandex-webmaster)

Managing sites through the Yandex Webmaster API v4.

- Indexing: history, samples, important URLs, archive export
- Search queries: top queries, history, extended analytics with filters
- Page recrawl: submitting URLs, status, quotas
- Links: broken internal, external (samples + history)
- Sitemaps: list, add, priority recrawl
- Diagnostics, SQI, feeds, PRO SERP export
- 24-hour TTL cache for session data

**Triggers (RU):**
- "проверь индексацию сайта" ("check site indexing")
- "покажи поисковые запросы" ("show search queries")
- "отправь на переобход" ("send for recrawl")

**Triggers (EN):**
- "yandex webmaster"
- "check site indexing"
- "recrawl url"

---

### [zoomkit](plugins/zoomkit/skills/zoomkit)

Introduction to ZoomKit, connecting, and working with its interface and the official API: billing, statistics reports and Yandex Direct settings.

- Explains the benefits, the published plans and the conditions under which the service pays for itself
- Key check and a warning when it is about to expire
- Service balance and the list of invoices already issued, filterable by status
- Two-step advertising statistics reports with waiting for readiness
- Bid rules and detailed link-check results for Yandex Direct
- List of the account's campaigns through the API, with the last sync date and a bid-management flag
- A hint for labeling unclear agency accounts through the "Comment" field
- Preview and a mandatory `--confirm` for changes
- Clear connection instructions when the key is missing
- Explicit protection against nonexistent methods: API 1.9.0 cannot issue invoices or create payment links

**Triggers (RU):**
- "проверь баланс ZoomKit" ("check the ZoomKit balance")
- "покажи счета ZoomKit" ("show ZoomKit invoices")
- "сделай отчёт ZoomKit" ("make a ZoomKit report")
- "что умеет ZoomKit и сколько стоит" ("what ZoomKit can do and how much it costs")
- "какие проекты подключены в ZoomKit" ("which projects are connected in ZoomKit")
- "покажи кампании клиента в ZoomKit" ("show the client's campaigns in ZoomKit")
- "не могу найти кабинет в ZoomKit" ("I can't find the account in ZoomKit")
- "найди кампании без проверки ссылок" ("find campaigns without link checking")
- "проверь правила ставок ZoomKit" ("check ZoomKit bid rules")

**Triggers (EN):**
- "zoomkit api"
- "zoomkit balance"
- "zoomkit invoices"
- "zoomkit pricing"

---

### [telegraph-publisher](plugins/telegraph-publisher/skills/telegraph-publisher)

Publishing pages to Telegraph with media support.

- Creating/editing pages through the API
- Support for images by URL and YouTube embeds
- For permanent images and diagrams, prefers GitHub + jsDelivr over the unreliable Telegraph upload
- For GitHub, recommends a separate public media repo and a separate fine-grained PAT with only `Contents: Read and write` on that repo
- Keeps the `Telegraph path -> assets` link in a manifest for later cleanup
- Auto-split of long materials into a series of pages
- Account management (creation, binding to a browser)

**Triggers (RU):**
- "опубликуй в Telegraph" ("publish to Telegraph")
- "создай страницу в Telegraph" ("create a Telegraph page")
- "telegraph публикация" ("Telegraph publication")

**Triggers (EN):**
- "publish to Telegraph"
- "create Telegraph page"
- "telegraph publish"

---

### [crawl4ai-seo](plugins/crawl4ai-seo/skills/crawl4ai-seo)

SEO site crawler based on Crawl4AI.

- Site inventory: URL, status, title, H1, meta, canonical, word count
- On-page audit: duplicate titles, empty meta, broken canonicals, thin content
- Internal linking analysis: orphan pages, weakly linked pages, link graph
- Navigation audit: breadcrumbs, menu consistency, weak hubs
- Landing page comparison and competitor analysis
- Works together with yandex-search-api, yandex-metrika, yandex-webmaster

**Triggers (RU):**
- "аудит сайта" ("site audit")
- "проверь перелинковку" ("check internal linking")
- "навигационный аудит" ("navigation audit")

**Triggers (EN):**
- "site audit"
- "internal linking audit"
- "seo crawl"

---

### [telegram-channel-parser](plugins/telegram-channel-parser/skills/telegram-channel-parser)

Parsing public Telegram channels through the web preview (t.me/s/).

- Channel posts with metrics (views, reactions, forwards)
- Digest across several channels for a period
- Top posts (share leaderboard), search, posting schedule
- Comparison table of channels
- Cache-first approach, zero config, no API keys

**Triggers (RU):**
- "парсинг телеграм канала" ("parse a Telegram channel")
- "дайджест каналов" ("channel digest")
- "анализ канала" ("channel analysis")

**Triggers (EN):**
- "telegram channel"
- "telegram digest"
- "telegram analytics"

---

### [x-research](plugins/x-research/skills/x-research)

Research on X/Twitter through the xAI Grok API with the x_search tool.

- Digest of subscriptions by account category
- Analysis of posts, threads and discussions (tone, arguments, community reaction)
- Trending topics by interest with sentiment scoring
- Free-form search with ideas for Telegram posts
- Session artifact store (live-first, immutable snapshots)

**Triggers (RU):**
- "дайджест твиттера" ("Twitter digest")
- "что в твиттере по [теме]" ("what's on Twitter about [topic]")
- "анализ твита" ("tweet analysis")

**Triggers (EN):**
- "x research" / "twitter research"
- "x digest" / "trending x"
- "analyze tweet"

---

### [github-pages-publisher](plugins/github-pages-publisher/skills/github-pages-publisher)

Publishing static artifacts to GitHub Pages.

- Deploying into the `YYYY/YYYY-MM/page-slug/` structure
- Viewport validation before publishing (desktop 1440px, mobile 375px)
- Title slugification, automatic `index.html` entrypoint
- Fine-grained GitHub token for safe pushes

**Triggers (RU):**
- "опубликуй на GitHub Pages" ("publish to GitHub Pages")
- "задеплой страницу" ("deploy the page")
- "залей артифакт" ("upload the artifact")

**Triggers (EN):**
- "publish to GitHub Pages"
- "deploy static page"
- "push artifact to Pages"

---

### [sourcecraft-publisher](plugins/sourcecraft-publisher/skills/sourcecraft-publisher)

Publishing static artifacts to SourceCraft Sites (Yandex, works in Russia).

- Deploying into the `YYYY/YYYY-MM/page-slug/` structure
- Automatic creation of `.sourcecraft/sites.yaml`
- OAuth2 token for pushing to SourceCraft
- Mirror of `github-pages-publisher` for Russian infrastructure

**Triggers (RU):**
- "опубликуй на SourceCraft" ("publish to SourceCraft")
- "задеплой на сорскрафт" ("deploy to SourceCraft")
- "залей в российское зеркало" ("upload to the Russian mirror")

**Triggers (EN):**
- "publish to SourceCraft"
- "deploy to SourceCraft Sites"

---

### [reddit-skill](plugins/reddit-skill/skills/reddit-skill)

Reading Reddit without keys through the agent's available web tool: public JSON
for posts and comments, RSS for listings. Shell scripts with a cache and the OAuth2 API are
a fallback and a base for automation.

- Built-in `fetch` / `open` first, if available; no app key checks
- RSS subscriptions are stored in [config/subscriptions.txt](plugins/reddit-skill/skills/reddit-skill/config/subscriptions.txt); no browser session needed
- `REDDIT_RSS_MODE=1`: top of a subreddit, search and user posts without an app or keys
- A single post and the available comments through the public `.json` in the same mode
- Automatic fallback to RSS when API authorization is refused; scores and comment counts stay unknown
- Direct Reddit OAuth2 API calls through curl (no PRAW or Python dependencies)
- Automatic mode choice: app-only (`client_credentials`) for read, user (`password`) for write/me
- Cache-first: tokens, users, subreddits, listings
- Double safeguard for write: `REDDIT_ENABLE_WRITE=1` + `--confirm` (without the flag it is a dry run)
- Rate limiting from headers (`x-ratelimit-*`, `Retry-After`)
- 14 operations: profiles, user posts/comments, top/popular subreddits, search, submission by URL/id, post_create, comment_reply, subscribe/unsubscribe

**Triggers (RU):**
- "посты reddit" ("Reddit posts")
- "комментарии reddit" ("Reddit comments")
- "парсинг reddit" ("parse Reddit")

**Triggers (EN):**
- "reddit api"
- "reddit subreddit"
- "reddit user"

---

### [knowledge-compiler](plugins/knowledge-compiler/skills/knowledge-compiler)

Compiling books, PDF/EPUB/TXT/Markdown and long applied sources into personal Claude Code skills.

- Builds not a summary but a map of applicable ideas: concepts, decision rules, playbooks, anti-patterns
- Suits technical, management, marketing, product, educational and internal materials
- Produces `source-map.json` and `knowledge-manifest.json` to link claims to source segments
- For EPUB, reads OPF metadata and `toc.ncx`/`nav.xhtml`, then uses the exact table of contents as the basis for segmentation
- Scouts the structure through a small `outline-scout.json`, first/last excerpts and candidate lines without loading the whole text
- Python scripts run through `uv run --script`, dependencies are declared in the scripts themselves
- Caches extracted text and segments in `cache/jobs/<job-id>/`; assembles the finished skill in `dist/<skill-name>/`
- Checks the result with `quality_gate.py`: required files, `SKILL.md` length, JSON and long verbatim matches
- OCR, a publishing profile and a Go accelerator are in the backlog

**Triggers (RU):**
- "сделай скилл из книги" ("make a skill from a book")
- "преврати PDF в навык" ("turn a PDF into a skill")
- "собери карту знаний" ("build a knowledge map")

**Triggers (EN):**
- "book to skill"
- "compile knowledge"
- "create skill from PDF"

---

### [perplexity-search](plugins/perplexity-search/skills/perplexity-search)

Search and research through the Perplexity API in POSIX shell.

- Search API: raw ranked results with snippets, up to 5 queries per paid call
- Agent API: answer with citations, choice of preset and model (Sonar, GPT, Claude, Gemini, Grok)
- Deep research in background mode with polling and `--resume` by response id
- Extracting the content of specific URLs through the `fetch_url` tool
- Structured output by JSON Schema
- Cache-first: the cache key is the request body itself; large results go to `cache/` and are read with grep
- Source profiles in `.env`: domains + freshness + extraction depth with one flag
- Offline tests: `.env` parsing, request body building, answer rendering, CLI contract, HTTP layer on a loopback mock

**Triggers (RU):**
- "найди в интернете" ("search the internet")
- "что пишут про [тему]" ("what is being written about [topic]")
- "глубокое исследование" ("deep research")

**Triggers (EN):**
- "perplexity search"
- "web search with sources"
- "deep research"

---

## Repository structure

```
polyakov-claude-skills/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace configuration
├── plugins/
│   ├── docx-contracts/       # Plugin for .docx
│   ├── scrapedo-web-scraper/ # Plugin for scraping
│   ├── agent-deck/           # Plugin for agents
│   ├── genome-analizer/      # Plugin for genome analysis
│   ├── ssh-remote-connection/# Plugin for SSH
│   ├── yandex-wordstat/      # Plugin for the Wordstat API
│   ├── codex-review/         # Plugin for cross-agent review
│   ├── fal-ai-image/         # Plugin for image generation
│   ├── yandex-search-api/    # Plugin for the Yandex Search API
│   ├── yandex-direct/        # Statistics and management for Yandex Direct
│   ├── yandex-metrika/       # Yandex Metrica reports, segments and access
│   ├── yandex-webmaster/     # Plugin for the Yandex Webmaster API
│   ├── zoomkit/               # Plugin for the ZoomKit API
│   ├── telegraph-publisher/  # Plugin for publishing to Telegraph
│   ├── crawl4ai-seo/         # Plugin for SEO crawling
│   ├── telegram-channel-parser/ # Plugin for parsing Telegram channels
│   ├── x-research/              # Plugin for X/Twitter research
│   ├── github-pages-publisher/  # Plugin for publishing to GitHub Pages
│   ├── sourcecraft-publisher/   # Plugin for publishing to SourceCraft Sites
│   ├── reddit-skill/            # Plugin for the Reddit API
│   ├── knowledge-compiler/      # Plugin for compiling sources into skills
│   └── perplexity-search/       # Plugin for search and research through the Perplexity API
└── README.md
```

---

## License

MIT
