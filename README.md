# App Store Top Charts by country: Top Free, Top Paid and Top Grossing iOS rankings as JSON

Read Apple's iOS App Store top charts for any storefront: Top Free, Top Paid and Top
Grossing for iPhone and iPad, plus what just launched (`new`, `new-free`, `new-paid`), in
the overall chart or any of 24 categories. Each row has the rank, app id, bundle id,
developer, price, genre and release date, and the rank change since your last run. No
Apple developer account, no API key from Apple.

This repo is the quick start for the hosted
[**App Store Top Charts Scraper**](https://apify.com/glitchbound/app-charts-scraper) on
Apify: one HTTP call from the shell, a standard-library Python script that writes a CSV,
and the setup for using it as a tool in Claude, ChatGPT or Cursor. For Android, see the
sister repo [google-play-top-charts-api](https://github.com/danielhagever/google-play-top-charts-api).

- [Quick start: one curl](#quick-start-one-curl)
- [Quick start: Python](#quick-start-python)
- [Use it from an AI agent (MCP)](#use-it-from-an-ai-agent-mcp)
- [Input](#input) · [Output](#output)
- [Tracking rank over time](#tracking-rank-over-time)
- [FAQ](#faq)

## Quick start: one curl

Get a free Apify API token at
[console.apify.com/settings/integrations](https://console.apify.com/settings/integrations),
then:

```bash
export APIFY_TOKEN=apify_api_...
curl -s -X POST \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"countries": ["us"], "charts": ["top-free", "top-grossing"], "categories": ["all"], "limit": 10}' \
  "https://api.apify.com/v2/acts/glitchbound~app-charts-scraper/run-sync-get-dataset-items"
```

The response is a JSON array, one object per charting app.

## Quick start: Python

[`example.py`](example.py) uses only the standard library and writes a CSV:

```bash
python3 example.py --country us --chart top-free --chart top-grossing --limit 3
python3 example.py inputs/games-four-markets.json
```

A real run of the first command:

```
6 chart rows written to charts.csv
  US top-free #1: Vinted: Pre-loved marketplace (Vinted Limited)
  US top-free #2: Muse from Meta (Meta Platforms, Inc.)
  US top-free #3: TikTok Pro - Events (TikTok Ltd.)
  US top-grossing #1: ChatGPT (OpenAI OpCo, LLC)
  US top-grossing #2: YouTube (Google)
```

## Use it from an AI agent (MCP)

Add this as a connector in Claude or ChatGPT:

```
https://mcp.apify.com?tools=glitchbound/app-charts-scraper
```

In Cursor or VS Code, the same address goes in `mcp.json`:

```json
{ "mcpServers": { "apify": { "url": "https://mcp.apify.com?tools=glitchbound/app-charts-scraper" } } }
```

The first call asks you to sign in to Apify or takes an API token. Then "what are the top
grossing games in Japan right now?" is one tool call.

## Input

| Field | What it does |
|---|---|
| `countries` | Two-letter storefront codes, lowercase: `us`, `gb` (not `uk`), `jp`, `in`, `il`. Any App Store storefront works. |
| `charts` | `top-free`, `top-paid`, `top-grossing`, the same three with `-ipad`, and `new`, `new-free`, `new-paid` |
| `categories` | `all`, or one of 24: `games`, `finance`, `education`, `health-fitness`, `productivity`, `shopping` and the rest |
| `limit` | Apps per chart, up to 100, which is Apple's maximum |
| `includeDetails` | Adds star rating, rating count, version, last update, size and age rating to every row, from one extra request per country |
| `trackRankChanges` | On by default: every row says where the app stood the last time your account read that chart |
| `maxItems` | Ceiling for the whole run |

Ready-made inputs are in [`inputs/`](inputs). A storefront Apple rejects (`uk` instead of
`gb`) comes back as an unbilled error row naming it, not as an empty result.

## Output

One row per charting app. Two real rows from the run above are in
[`sample-output.json`](sample-output.json); one of them, abridged:

```json
{
  "rank": 1,
  "country": "US",
  "chart": "top-grossing",
  "category": "all",
  "appId": "6448311069",
  "bundleId": "com.openai.chat",
  "title": "ChatGPT",
  "developer": "OpenAI OpCo, LLC",
  "price": 0,
  "isFree": true,
  "genre": "Productivity",
  "releaseDate": "2023-05-18T00:00:00-07:00",
  "url": "https://apps.apple.com/us/app/chatgpt/id6448311069?uo=2",
  "chartedAt": "2026-10-26T00:57:11Z"
}
```

`appId` is Apple's numeric id and `bundleId` the package name; both are stable across runs
and storefronts.

## Tracking rank over time

Rankings are interesting over time. Schedule a daily run on Apify (or call it from cron) and
from the second run on every row also carries `previousRank`, `rankChange` (positive means
it climbed), `newInChart` and `previousChartedAt`, so the dataset is a rank tracker rather
than a pile of snapshots to diff.

## FAQ

**Is there an official App Store top charts API?**
Apple publishes its top charts as public feeds (the iTunes RSS charts). This reads exactly
those, so the ranks are Apple's own, with no key and no login.

**How do I compare countries?**
Pass several `countries` in one run; each country, chart and category combination comes
back in the same dataset.

**Is the Mac App Store included?**
No. Apple serves a Top Free Mac Apps feed in the same format, but its Top Paid and Top
Grossing Mac feeds came back empty, so the Mac is left out rather than offered as one chart
of three.

**What does it cost?**
It is paid per chart row; rows for charts that could not be read are not charged. The
current rate for your plan is on the [Actor page](https://apify.com/glitchbound/app-charts-scraper).

## Related

- [Google Play Top Charts Scraper](https://apify.com/glitchbound/google-play-charts-scraper): the Android charts, 130 countries, with Trending; quick start in [google-play-top-charts-api](https://github.com/danielhagever/google-play-top-charts-api).
- [App Store & Google Play Scraper](https://apify.com/glitchbound/app-store-scraper): one app's ratings, price and metadata in both stores.
- [App Store & Google Play Reviews Scraper](https://apify.com/glitchbound/app-reviews-scraper): every review for an app, both stores.
- [ASO Keyword Rank Tracker](https://apify.com/glitchbound/aso-keyword-rank-tracker): where an app ranks for a keyword in both stores.

Questions or a chart that looks wrong? Open an issue here or on the Actor page.

## License

MIT
