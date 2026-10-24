# web scraping excel power query: When Excel's Built-in Tool Is Enough, When You Need a Scraping API, and How to Wire Them Together

Power Query can handle a lot of web scraping straight out of Excel — but only for pages that serve plain, static HTML. The moment your target site renders content with JavaScript, sits behind Cloudflare, or starts blocking your IP after the tenth refresh, you need to route your requests through a scraping API. This guide covers both halves: how to get the most out of Power Query itself, and how to plug ScraperAPI into it when Excel hits its ceiling.

> **Quick picks**
> - Static HTML tables (public data pages, Wikipedia, listings) → Power Query alone, zero cost.
> - JavaScript-heavy sites, Google/Amazon/LinkedIn data, or anything that starts returning 403s → Power Query + ScraperAPI. The free trial gives you **5,000 credits for 7 days, no credit card required** — enough to test your real targets before paying anything. 👉 [Start the free trial and test your target URLs](https://www.scraperapi.com/signup/?fp_ref=coupons)
> - If you do end up on a paid plan, annual billing takes 10% off automatically, and a 10% code for first-month signups has been circulating alongside it — more on what's verifiable below.

## What Power Query does well — and how to use it properly

Power Query is Excel's built-in web scraper, and for static pages it's genuinely good. Microsoft has quietly retired the old Web Queries (.iqy files) — they depended on Internet Explorer rendering, couldn't parse JSON, and failed immediately on any page requiring cookies or custom headers. Power Query ("Get & Transform") replaced all of that with automatic table detection and JSON support.

The basic workflow takes about two minutes:

1. Go to **Data → Get Data → From Web**.
2. Paste the URL.
3. Excel analyzes the page and shows detected tables in the Navigator window.
4. Pick a table, clean it up in the Power Query Editor if needed, then **Close & Load**.

The killer feature is refreshability. Your query isn't a one-time copy-paste; click **Refresh All** later and Excel re-pulls the same table from the same URL. If the source page updates, your spreadsheet updates.

One thing most tutorials skip: the standard "From Web" dialog box gives you no way to add request headers. The moment a site requires a User-Agent (or an API key in an Authorization header), you'll get a 403 Forbidden with no obvious fix. The solution is the **Advanced Editor**: open it from the Power Query Editor, and pass a Headers record through the `Web.Contents` function, like:


= Json.Document(Web.Contents("https://api.example.com/data",
    [Headers=[Authorization="Bearer YOUR_KEY", Accept="application/json"]]))


Adding a browser-like User-Agent string (something starting with "Mozilla/5.0…") to mimic a real browser fixes a surprising number of basic 403 errors on public sites. Headers passed this way are the only reliable method — the UI dialog simply won't do it.

## Where Power Query falls apart

Power Query has three hard limits, and knowing them upfront saves hours of debugging.

**First: no JavaScript.** Excel cannot execute JavaScript. Modern sites built with React, Vue, or Angular serve a nearly empty HTML shell — you'll pull in a page containing a single empty `<div id="root"></div>` and wonder where your table went. There's no setting to change this; Power Query fetches raw HTML and that's the end of its involvement. The workaround is routing the URL through an external rendering service that runs a headless browser and hands Excel the finished HTML or JSON.

**Second: no proxy rotation.** Excel sends every request from your own IP address. Pulling a stock price once a day? Fine. Refreshing a 5,000-row product price sheet every hour? Your office IP gets blocked long before lunch. Power Query has no built-in way to rotate proxies or distribute requests.

**Third: no anti-bot handling.** CAPTCHAs, rate limiting, Cloudflare challenges — Power Query just fails, and you get an error cell instead of data.

If your targets dodge all three traps, stop reading and use Power Query as-is. If any of them apply, keep going.

## How to plug ScraperAPI into Power Query

ScraperAPI works as a middle layer: you send the URL you want to scrape to its endpoint along with your API key, and it returns the page's HTML — after handling IP rotation (residential and datacenter proxies across 50+ countries), CAPTCHA solving, headless browser rendering, and automatic retries on failed requests. From Power Query's perspective, you're just calling a very cooperative URL.

The setup fits naturally into the `Web.Contents` pattern from the previous section, since the API key is passed as a URL parameter rather than a header. A minimal request looks like:


= Web.Contents("https://api.scraperapi.com/?api_key=YOUR_KEY&url=https%3A%2F%2Fwww.example.com%2Fpage")


Two parameters matter most for Excel users:

- `render=true` — forces JavaScript rendering, which is what you need for React/Vue sites that Power Query can't see through on its own.
- `country_code` — pins the request to a specific country, useful when the site shows different prices or content by region.

ScraperAPI also exposes structured data endpoints that return clean, pre-parsed JSON for Amazon, Google Search, Google Shopping, and Walmart — prices, titles, ratings — so you skip the HTML-parsing step entirely and load the JSON straight into a table with `Json.Document`. For scheduled bulk jobs beyond what a spreadsheet should reasonably hold, there's DataPipeline, a no-code option for recurring collections, plus a Crawler for following links across a whole domain.

One more tool worth knowing about if your target doesn't fit a pre-built endpoint: the **AI Parser**. You define a schema for the fields you want, and it extracts structured data from a page automatically. The billing rules are easy to misread, so here they are plainly — your **first parser is free forever**, no credits charged, and parsing URLs costs nothing beyond the normal scrape. But creating each *additional* parser costs 30,000 credits, and so does adding or modifying fields later. The free plan can create a parser but can't update it; paid plans hold up to 5 parsers at once (Enterprise unlimited). For an Excel workflow, one free parser refreshing a fixed set of fields is often all you need.

Practical notes worth knowing before you start: every request has a 50MB size limit, and ScraperAPI's own docs recommend setting a 70-second timeout in your client for the best success rates on difficult domains. If your query times out in Power Query's default window, that timeout setting is the first thing to check. Integration effort is genuinely low — you're replacing one HTTP request URL with another, no new framework to learn. 👉 [Sign up and grab your API key to test your first query](https://www.scraperapi.com/signup/?fp_ref=coupons)

## ScraperAPI plans and pricing: the full picture

ScraperAPI bills by monthly API credits across eight tiers. All paid plans include the same core feature set — JavaScript rendering, premium proxies, CAPTCHA and anti-bot handling, JSON auto-parsing, custom headers and sessions, automatic retries — the differences are volume, concurrency, and geotargeting granularity.

| Plan | Monthly price | Annual (per-month) | Credits/month | Concurrency | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Start free](https://www.scraperapi.com/signup/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [See Hobby plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [See Startup plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [See Business plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [See Scaling plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [See Professional plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [See Advanced plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Annual billing takes 10% off across every tier, applied automatically — no code needed. There's also a documented Student plan for eligible users.

A few plan-level details that don't show up in the pricing headline:

- **Hobby and Startup geotargeting is US & EU only.** Country-level targeting anywhere in the world starts at Business ($299/month). If you need Southeast Asian or Latin American localized data, the cheap tiers won't do it.
- **Hobby and Startup limit your analytics dashboard to the last 30 days** of data. Business and above get unlimited history — a small thing until you're trying to compare this month's success rates against last quarter's.
- **Pay-as-you-go overage only exists on Scaling and above** (plus Professional, Advanced, Enterprise). On Hobby through Professional monthly plans, you can opt into Auto-Upgrade instead, which bumps you to the next tier automatically when credits run out — usually at a better price per credit.
- **Credits don't roll over.** They reset each billing period, so buying a bigger plan "to bank credits" doesn't work.
- **Cancel anytime** from the dashboard, no cancellation fee, and there's a **7-day no-questions-asked refund policy** if you're unhappy after paying.
- Exceeding your concurrency limit returns a 429 error with no charge; the request is rejected rather than queued.

## The credit math most reviews get wrong

Here's the part that matters more than the sticker price: a credit is the cost of a *standard* page request, and many popular targets cost far more than one credit.

The official rates: a standard page costs 1 credit, Amazon pages cost 5, Google and Bing (including their subdomains) cost 25, LinkedIn costs 30. Sites behind bot protection like Cloudflare, Datadome, or PerimeterX add 10 credits per request when the protection is bypassed. On top of the domain cost, JavaScript rendering multiplies the base price — third-party reviews put the render multiplier at roughly 5–10x, so an Amazon page fetched with `render=true` runs somewhere between 25 and 50 credits rather than 5.

That's where the Hobby plan's 100,000 credits get misleading. Pointed at Amazon product pages with JS rendering enabled, 100,000 credits translates to roughly 2,000–4,000 successful requests per month — not 100,000. The same credits against plain static HTML pages get you the full 100,000 requests. Same plan, order-of-magnitude difference in real output. (For what it's worth, DataPipeline batch jobs now bill at the same rates as standard scraping — ScraperAPI dropped its old, steeper DataPipeline pricing in 2026 — so the principle holds across both.)

Two tools keep this under control:

1. **Domain Cost Estimator** in the dashboard shows the exact credit cost for any specific URL before you build anything around it.
2. The `max_cost` parameter sets a hard ceiling per request, so one misconfigured scrape can't silently burn a week's budget.

Before committing to any plan, check your actual target URLs with the estimator and divide your plan's credits by the real per-request cost. 👉 [Check your target URLs against the credit cost table](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Which plan actually fits an Excel workflow

For the typical "web scraping excel power query" use case — refreshing a spreadsheet with data from a handful of stubborn websites — you don't need much volume.

**Free plan (1,000 credits/month, 5 concurrent):** enough to test whether ScraperAPI can reach your specific targets at all. Use it before buying anything. During your first 7 days you also get 5,000 trial credits for larger-scale testing.

**Hobby ($49/month, 100,000 credits):** the natural fit for personal Excel dashboards and small recurring reports. At 1 credit per standard page, that's thousands of daily refreshes with room to spare; even against Amazon-class targets it covers a serious single-user workflow. The constraints to remember are the US/EU-only geotargeting and the 30-day analytics window.

**Startup ($149/month, 1,000,000 credits):** where you land if the spreadsheet is feeding regular competitive monitoring — price tracking, SERP checks — and the Hobby tier's geotargeting is too narrow. This tier opens up country-level positioning fully.

**Business and above ($299+/month):** production workloads and agencies running many scheduled jobs. At this point you're probably better served by DataPipeline's batch and scheduled-task model than by refreshing spreadsheets manually, and Pay-as-you-go overage becomes available.

If your scraping needs stay entirely within static HTML tables, none of these tiers are for you — Power Query alone does the job for free, and that's the honest answer for a large share of searches on this topic.

## FAQ

**Can Excel Power Query scrape JavaScript-rendered websites?**
No. Power Query fetches raw HTML and cannot execute JavaScript, so React, Vue, and Angular sites return empty shells. You need an external rendering service — ScraperAPI's `render=true` parameter handles this and returns the fully rendered page to Excel.

**How much does web scraping cost per month?**
Anywhere from $0 to $49 for most individual Excel workflows: Power Query is free and handles static pages, and ScraperAPI's Hobby plan at $49/month (or $44.10 on annual billing) covers typical blocked or dynamic targets. Heavier production setups run $149–$475/month depending on volume and geotargeting needs.

**Are there discount codes?**
The only discount you can rely on without a credit card risk is the automatic 10% off for annual billing — it applies to every paid tier with no code. Independently published reviews have also verified a code called **START10**, good for 10% off the first month of any paid plan; it's reported to stack with the annual discount. The "50% off" and "28% off" codes floating around coupon aggregator sites are largely unverified and routinely fail at checkout, so don't build your budget around them.

**Can I add API keys and custom headers in Power Query?**
Yes, but only through the Advanced Editor using the `Web.Contents` function's Headers record — the standard From Web dialog offers no header options. ScraperAPI sidesteps this anyway by accepting the API key as a URL parameter.

**What happens if I run out of credits mid-month?**
On Hobby through Professional monthly plans, you can enable Auto-Upgrade to move to the next tier automatically, or contact support for a custom plan. Scaling and higher tiers include Pay-as-you-go, letting you keep scraping at a fixed per-credit rate up to a monthly spending cap you set yourself — and it's a spending limit, not a pre-charge.

## The bottom line before you commit

Match the tool to the page, not the other way around. Static HTML tables that Power Query can already read: use it, spend nothing. One or two dynamic or protected sites feeding a personal spreadsheet: the Hobby plan at $49/month almost certainly covers it, with the free 7-day trial confirming your specific targets work before you pay a cent. Country-level data needs or recurring high-volume pulls: Startup or Business, respectively.

Two last checks before clicking buy: run your real URLs through the Domain Cost Estimator so the credit math is based on your targets rather than the 1-credit headline number, and remember Hobby/Startup are US/EU-only if geotargeting matters to you. If both check out, 👉 [start with the 7-day free trial — no credit card needed](https://www.scraperapi.com/signup/?fp_ref=coupons) — and you'll know within a week whether it fits your workflow.
