# news scraper n8n: build an automated news pipeline with ScraperAPI, plus what each plan really costs per scrape

Most "news scraper n8n" tutorials stop at the workflow screenshot. They show you an RSS trigger, a scraping node, maybe an AI summary step — and then skip the part that actually determines whether your automation survives week two: what the scraping service costs when you run it every day, and which plan won't run out of credits by Thursday.

This article covers both. First, the realistic ways to scrape news inside n8n and where an official ScraperAPI node fits. Then the full, current ScraperAPI plan lineup, followed by the math most tutorials leave out — how credit multipliers turn a "100,000 credits" plan into anywhere from 1,300 to 100,000 actual scrapes, and which hidden limits (geo restrictions, concurrency caps, pay-as-you-go rules) matter for a news workflow specifically.

## Quick pick before the details

If you just want the short version:

- For a personal news digest or monitoring workflow running a few dozen feeds a day, **Hobby ($49/mo, 100,000 credits) is almost always enough** — and it's the cheapest paid tier.
- You can test everything on the **7-day trial with 5,000 credits and no credit card**, or the free plan that gives you 1,000 credits and up to 5 concurrent connections on signup.
- Annual billing takes 10% off every paid plan, so Hobby drops to $44.10/mo if you're committed.

👉 [Check ScraperAPI's plans and start the 7-day free trial](https://www.scraperapi.com/?fp_ref=coupons)

## Your options for scraping news in n8n

There are four realistic routes, and they differ more in maintenance burden than in capability.

**RSS feeds plus a scraping node.** The most common pattern in n8n's template library: pull headlines from RSS feeds, then scrape each article's full text. One popular template pairs RSS.app with a scraping step and saves clean Markdown to Google Drive for newsletter curation. The limitation is obvious — RSS only covers sites that publish feeds, and many news sites truncate or omit them.

**Firecrawl workflows.** Several n8n templates use Firecrawl to convert scraped articles to Markdown before storage. This works well for LLM pipelines, and the free template tier gets hobbyists started.

**Bright Data via MCP.** A newer approach connects n8n to Bright Data's MCP server and lets an AI agent call scraping tools conversationally. One widely shared build used it for a dev news aggregator. Worth knowing its stated limitation: paywalled sources like Bloomberg or WSJ won't scrape without account credentials and authentication cookies, which the workflow itself doesn't handle.

**Marketplace actors.** Apify runs a dedicated "n8n news scraper" actor that extracts headlines, full text, and metadata by keyword or region, connecting to n8n for media monitoring. Community nodes exist for specific sources too — there's one built for Naver News, the Korean aggregator, powered by an Apify actor underneath.

What all of these share is a per-request billing model of some kind. Where ScraperAPI separates itself for n8n users is the integration depth: an official community node (not a generic HTTP workaround) plus a hosted MCP server, with proxy rotation, CAPTCHA handling, and JavaScript rendering handled server-side so your workflow stays simple.

## The official ScraperAPI n8n node: what it does and how to set it up

ScraperAPI ships an official n8n community node, and it's a real node with four resource types, not a thin wrapper around one HTTP call.

**Installation takes about two minutes:**

1. In your n8n workspace, open the Nodes panel and search for "ScraperAPI."
2. Click Install. The node appears in your workflow palette.
3. Grab your API key from the ScraperAPI dashboard.
4. In the node, click "Credential to connect with," create a new credential, paste the key, and save — n8n tests it automatically.

The four resources cover different jobs:

- **API** — sends a GET request to any URL and returns the page content, handling proxies, browsers, and CAPTCHAs along the way. This is what you'd point at news article URLs.
- **Structured Data Endpoints (SDEs)** — purpose-built endpoints that return clean parsed JSON for major platforms. The list includes **Google News**, alongside Amazon product/search, Google Search/Jobs/Shopping/Maps, eBay, Walmart, and Redfin. If your workflow tracks search-driven news coverage, this endpoint returns structured data without manual parsing.
- **Crawler** — starts from a seed URL and follows links across multiple pages, streaming results to a webhook you provide. Useful for crawling a news site's section pages rather than individual articles.
- **AI Parser** — you build a reusable parser from a few example URLs, then extract structured JSON from any page with the same layout. Handy when a news site has a consistent article template.

The AI agent setup is documented too: a Chat Message Received trigger, an AI Agent node with a chat model (OpenAI or similar) and Simple Memory connected, and the ScraperAPI node attached as the agent's tool. You write a system prompt describing what the agent should do, and it decides when to scrape. For a "monitor topics and summarize" news workflow, this is the least-code path.

There's a third integration route if you prefer MCP: ScraperAPI hosts an MCP server at `mcp.scraperapi.com/mcp` that works with n8n's MCP Client Tool. Configure it with HTTP Streamable transport and Bearer Auth using your API key as the token, include all tools or a subset, and an agent can call scraping functions directly.

One detail that trips people up: the node's parameter names use camelCase with a resource prefix (`countryCode`, `outputFormat`), while the raw HTTP API uses snake_case (`country_code`, `output_format`). If you copy example payloads from the HTTP docs into the node, they won't match.

## Full plan comparison

Here's every plan currently listed on the official pricing page:

| Plan | Monthly price | Annual (10% off) | Credits | Concurrent threads | Get it |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10/mo | 100,000 | 20 | [Get Hobby](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10/mo | 1,000,000 | 50 | [Get Startup](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10/mo | 3,000,000 | 100 | [Get Business](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling (Most popular) | $475 | $427.50/mo | 5,000,000 | 200 | [Get Scaling](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50/mo | 10,500,000 | 300 | [Get Professional](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50/mo | 21,500,000 | 500 | [Get Advanced](https://scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://scraperapi.com/pricing/?fp_ref=coupons) |

Every paid plan includes the same core feature set: JS rendering, premium residential and mobile proxies, JSON auto parsing, rotating proxy pools, custom headers and sessions, CAPTCHA and anti-bot detection, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. You're choosing a credit budget and concurrency ceiling, not unlocking features as you climb.

## The real cost: credits vs. requests

This is the part most pricing pages gloss over, and it changes which plan you should buy.

One credit is the cost of a standard page request — but a news scraper rarely sticks to standard requests. ScraperAPI's pricing FAQ is explicit: a standard page costs 1 credit, Amazon costs 5, Google and Bing (including subdomains) cost 25, and LinkedIn costs 30. Sites behind bot protection like Cloudflare, Datadome, or PerimeterX add 10 credits per request when bypassed. On top of that, enabling JavaScript rendering or premium proxies costs 10 credits per request, and the combinations stack — premium plus render is 25, ultra-premium plus render reaches 75.

For a news workflow, the practical takeaway: scraping a plain article page costs 1 credit, but if that page needs JS rendering (many modern news sites do), it's 10. And the moment your workflow hits Google News or Google Search results, each request burns 25 credits.

Put numbers on it. The same 100,000-credit Hobby plan delivers:

- **~100,000 scrapes** of static article pages
- **~10,000 scrapes** if every request needs JS rendering — roughly 330 per day
- **~4,000 scrapes** if you're hitting Google SERPs at 25 credits each

That's a 75× swing depending on configuration, which is why "how many articles can I scrape per day?" has no single answer. Two useful safeguards exist in the dashboard: the Domain Cost Estimator shows the exact credit cost for any URL before you build the workflow around it, and a `max_cost` parameter caps what any single scrape can consume, so one misconfigured request can't drain your monthly pool.

Also worth noting for budgeting: `autoparse=true` returns parsed JSON for supported sites at no extra credit cost, and `outputFormat` can return markdown or text — both friendly to downstream AI steps in n8n without inflating your bill.

## Hidden limits that affect news workflows

The pricing grid doesn't show these, but they're in the FAQ and docs, and they matter.

**Geotargeting is restricted on the two cheapest plans.** Hobby and Startup support US & EU geotargeting only. Business and above unlock country-level targeting with any two-letter ISO code. If your news monitoring needs coverage from specific countries outside Europe, Hobby won't do it — that alone can force the jump to $299/mo.

**Analytics history is 30 days on Hobby and Startup.** Business and above keep unlimited dashboard history. For a monitoring workflow where you might want to look back at what you scraped months ago, this is a real constraint.

**Concurrency rejection, not queuing.** Exceeding your plan's thread limit returns a 429 and the request is rejected outright — it doesn't queue and it doesn't charge you. A news scraper polling 30 feeds on the free plan's 5-connection cap will feel this immediately. Hobby's 20 threads is comfortable for most personal setups.

**What happens when credits run out depends on your plan.** By default, scraping pauses until credits renew. On Hobby through Professional, you can opt into auto-upgrade, which moves you to the next plan automatically when credits exhaust. On Scaling and above, pay-as-you-go keeps scraping past your limit at a fixed per-credit rate, capped at a monthly spending limit you set. Annual plans can renew early for a fresh batch of credits. If you'd rather never wake up to a silently stopped workflow, note that PAYG — the option that keeps things running without changing plans — starts at Scaling ($475/mo).

**Parameter combinations have landmines.** In the node: `premium` and `ultraPremium` are mutually exclusive; `sessionNumber` can't combine with either; `retry404` is unavailable with premium proxies; and `keepHeaders` — which forwards your custom headers to the target site — only works through n8n's generic HTTP Request node, not the dedicated one, and is discarded when ultraPremium is on. ScraperAPI's own warning is worth repeating: overriding their header management often lowers success rates, so only set custom headers when a site specifically requires them.

Two more parameters quietly raise your per-request cost: `instructionSet` (browser actions like `click` or `wait_for_event`) and `waitForSelector` both implicitly enable rendering, and rendering is billed at 10 credits — so a workflow that looks like it's doing 1-credit requests can actually be spending 10. Easy to trigger accidentally.

## Which plan fits which news workflow

Matching budgets to actual jobs:

- **Personal digest, 50–200 articles/day, mostly static pages or light rendering:** Hobby. With rendering on for everything, 100,000 credits still buys about 10,000 scrapes a month — over 300 a day, well above what a personal news digest consumes. Without rendering, you're nowhere near the ceiling.
- **Team monitoring, multiple sources, some SERP-based tracking:** Startup ($149). The 1M credit pool absorbs Google News queries at 25 credits each — roughly 40,000 SERP calls per month even if every single request hit Google — while 50 threads keep parallel feed processing moving.
- **Country-specific coverage or unlimited analytics history:** Business ($299) is the floor. No cheaper plan offers non-US/EU geotargeting.
- **Production systems that must never pause:** Scaling ($475) adds pay-as-you-go overage, so an unexpectedly hot news cycle doesn't halt your pipeline mid-month.

Professional and above exist for high-volume commercial scraping; a news workflow alone rarely justifies $975+/mo unless it's feeding a product.

👉 [See current plan prices with the yearly discount applied](https://scraperapi.com/pricing/?fp_ref=coupons)

## FAQ

**Does the n8n node cost extra?** No. The ScraperAPI community node is free to install, and the hosted MCP server is included. You pay only for API usage against your plan's credits.

**Can I try it before paying?** Yes — a 7-day trial with 5,000 credits requires no credit card, and there's also a standing free plan with 1,000 credits, capped at 5 concurrent connections. The concurrency cap is small enough that a parallel feed-polling workflow will notice it; it's meant for testing, not running. If you need more testing credits, support can grant additional ones.

**Will it scrape paywalled news sites?** Not without your own credentials. Like the Bright Data MCP approach documented in public builds, paywalled sources require account authentication — ScraperAPI's `keepHeaders` can forward your custom headers and cookies (via the HTTP Request node), but the official docs caution this often reduces success rates, and it's incompatible with ultraPremium.

**What output formats does the node return?** `outputFormat` supports markdown and text on the standard API; CSV and JSON are available for autoparse-supported sites. Structured Data Endpoints accept json, csv, markdown, or text. For feeding an LLM summarization step in n8n, markdown output keeps the context clean.

**Does a failed request still burn credits?** Failed attempts retry automatically (automatic retries are included on all plans), and the `retry404` option re-attempts requests that return 404 — though that parameter can't be used alongside premium proxies.

## Before you commit

A few last checks that save money:

- Run your target news URLs through the Domain Cost Estimator before picking a plan. If your sources are mostly standard pages, Hobby is fine; if half of them sit behind Cloudflare, your effective capacity is a fifth of the headline number.
- If you're unsure whether you'll need non-US/EU geotargeting, start on Hobby with the yearly-off price in mind and upgrade only when a specific source demands it — upgrades happen from the billing page anytime.
- Skip auto-upgrade on your first month. It's opt-in for a reason; better to hit the credit wall once, see your real monthly consumption, then decide between upgrading and PAYG.

For most people searching "news scraper n8n," the sequence is: install the official node, grab an API key, run the trial on a Hobby-scale workflow, and only then decide between monthly and annual billing.

👉 [Start your 7-day trial with 5,000 free credits](https://www.scraperapi.com/?fp_ref=coupons)
