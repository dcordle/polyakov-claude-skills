---
name: scrapedo-web-scraper
description: |
  Web scraping through Scrape.do. Bypasses blocks and CAPTCHAs.
  Use AUTOMATICALLY when WebFetch fails with: 403, 401, 429,
  timeout, access denied, Cloudflare block.
---

# Scrape.do Web Scraper

Scrape web pages through the Scrape.do API. Use it when a plain fetch does not work (blocking, JavaScript).

## Usage

```bash
# Get the page text
python scripts/scrape.py https://example.com

# Get the HTML
python scripts/scrape.py --html https://example.com
```

## From Python

```python
from scripts.scrape import fetch_via_scrapedo

result = fetch_via_scrapedo('https://example.com')
if result['success']:
    print(result['content'])  # text
    # result['html'] is the original HTML
else:
    print(result['content'])  # error description
```

## Result

- **Success**: the page text (or HTML with `--html`)
- **Error**: a clear message (no token / limit reached / unavailable)

If an error comes back, the page is not reachable with this method.
