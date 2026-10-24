# Tripadvisor Review Scraper: How to Pull Hotel & Restaurant Reviews Without Getting Blocked, and What Each Plan Actually Costs

Scraping Tripadvisor reviews comes down to three workable routes: a no-code cloud scraper (Apify hosts the leading one at roughly $0.90 per 1,000 reviews), a general scraping API that fetches and renders pages for you (ScraperAPI is the option covered in this guide), or an open-source script you run yourself. Which one fits depends on whether you want a finished dataset or a pipeline you control — and how much you care about cost per review versus flexibility across other sites.

> **Quick pick:** if you're comfortable with a bit of Python and want one API that also handles Google, Amazon, and other targets later, ScraperAPI's Hobby plan ($49/month, 100,000 credits, 7-day free trial with 5,000 credits, no credit card) is the sensible starting point. 👉 [Check ScraperAPI's current plans and start the free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Your three options for a Tripadvisor review scraper

The market splits cleanly into three categories, and picking the wrong one is the most common (and most expensive) mistake in this niche.

**Dedicated cloud scrapers.** On Apify, the most popular Tripadvisor place actor has 14,433 users, a 4.93 rating and a 99.9% run success rate — but it deliberately does not return individual review text, only review tags and rating histograms. For review text you need the separate reviews actor, which runs $0.90 per 1,000 reviews (verified August 2026) with a 4.96 rating and 100% run success. Paste a listing URL, click start, download JSON/CSV/Excel.

**General scraping APIs.** You send a Tripadvisor URL to an endpoint, the service renders JavaScript, rotates proxies, solves CAPTCHAs, and hands you back the HTML (some add parsing on top). ScraperAPI works this way. It's more work than a dedicated actor — you parse the reviews yourself — but the same key later scrapes Google SERPs, Amazon, or any other site, and you control every parameter.

**Open-source scripts.** GitHub has several Tripadvisor review scrapers, including one built on Tripadvisor's internal GraphQL API with TLS fingerprint spoofing that avoids browser automation entirely. Free, but you own the proxy problem, and Tripadvisor's DataDome protection is exactly why paid tools exist.

One background fact that explains why this market exists at all: the official TripAdvisor Content API caps you at **5 reviews per location**. That's a product decision, not a technical limit, and it's useless for sentiment analysis or any real research — so everyone doing serious work ends up scraping.

## ScraperAPI plans and pricing

ScraperAPI prices by API credits. Every request consumes credits, and the amount depends on the target domain and the parameters you add — more on that in the next section, because it changes the math more than the plan price does.

| Plan | Monthly price | Annual (per month) | API credits | Concurrent threads | Geotargeting | Get started |
| --- | --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | — | [ Sign up free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | US & EU only | [ View Hobby plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | US & EU only | [ View Startup plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | 50+ countries | [ View Business plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | 50+ countries | [ View Scaling plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | 50+ countries | [ View Professional plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | 50+ countries | [ View Advanced plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | 50+ countries | [ Contact sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Annual billing takes 10% off every paid plan. Before any of that, there's a 7-day free trial with 5,000 credits and no credit card required, plus a permanent free plan with 1,000 credits monthly.

All plans include JS rendering, premium residential and mobile IPs, CAPTCHA handling, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. When you exceed your concurrency limit you get a 429 response — requests are rejected, not queued, and you're not charged for them. When you run out of credits, scraping pauses by default; Hobby through Professional plans can opt into auto-upgrade to the next tier, and Scaling and above can keep going with pay-as-you-go at a fixed per-credit rate.

## How to scrape Tripadvisor reviews with ScraperAPI

The core loop is simple: find the listing's location ID, request each reviews page through the API with rendering on, and parse the HTML.

The location ID is the number after `-d` in any Tripadvisor URL:


https://www.tripadvisor.com/Hotel_Review-g60745-d258705-Reviews-Hotel_Name.html
                                    ^^^^^^ location ID: 258705


Then it's a standard request with JS rendering enabled:

python
import requests
from bs4 import BeautifulSoup

response = requests.get(
    "https://api.scraperapi.com",
    params={
        "api_key": API_KEY,
        "url": "https://www.tripadvisor.com/Hotel_Review-g60745-d258705-Reviews-Hotel_Name.html",
        "render": "true",
    },
)
soup = BeautifulSoup(response.text, "html.parser")


Pagination works by rewriting the URL the same way Tripadvisor's own site does — replace `-Reviews-` with `-Reviews-or10-`, `-or20-`, and so on (10 reviews per page offset). Every review page gives you reviewer name, star rating, review text, date, and title in the HTML; fields like trip type, stay date, and management responses are on the page too but need more careful selectors.

What you get back is raw HTML, not structured JSON. That's the honest trade-off versus a dedicated actor, and it's worth knowing before you commit: in a recent independent benchmark of four scraping APIs on 500 Tripadvisor review pages, none of the providers returned structured JSON — all relied on rendering the page and parsing with CSS selectors.

Two practical tips from the docs that will save you money. First, the `country_code` parameter (useful if you want to see reviews as they appear to users in Germany or Japan) adds **zero extra credits**. Second, every response includes a `sa-credit-cost` header telling you exactly what that request cost, so you can verify your assumptions on the first request before running thousands.

👉 [Grab an API key and test it on your first hotel URL](https://www.scraperapi.com/?fp_ref=coupons) before committing to anything — the trial doesn't require a card.

## What a Tripadvisor review page actually costs in credits

This is the part most comparison articles skip, and it's where budgets get burned.

Tripadvisor isn't in ScraperAPI's special domain list, so a plain request costs the base **1 credit**. But a review page needs JavaScript rendering, and `render=true` adds **10 credits**. If the request trips an anti-bot bypass — Tripadvisor is known to use DataDome, and each bypass (Cloudflare, DataDome, PerimeterX, Akamai) costs an extra **10 credits** when triggered — a single rendered, protected page can run around 21 credits. ScraperAPI's own documentation says some domains may change protection over time and recommends confirming the exact cost per URL before scraping.

You can do that without burning trial credits:

- The **Domain Multiplier** tool in your dashboard looks up the credit cost of any URL
- The **API Playground** tests the URL and shows real credit usage
- An API endpoint (`/account/urlcost`) accepts your key, target URL, and all parameters, and returns the cost — so you can script the check

Run the math for the Hobby plan's 100,000 credits: at roughly 11 credits per rendered request, that's about 9,000 review pages a month. At Business scale (3,000,000 credits for $299), a rendered page works out near $1 per thousand. Compare that with the dedicated Apify reviews actor at $0.90 per 1,000 *reviews* — but note the units differ. One rendered page holds multiple reviews, so the per-review gap narrows; the price you're really paying ScraperAPI for is general-purpose flexibility and infrastructure you don't maintain.

The bigger cost trap applies to every tool: review counts per property follow a long-tail distribution. A neighborhood restaurant might have 40 reviews; a major city hotel can have tens of thousands. One documented example of a single venue returned 55,000 reviews — around $50 at the $0.90/1K rate, and scaling that to 100 flagship hotels becomes a five-figure spend. The fix, regardless of tool: use a date cutoff. Reviews older than about two years rarely change your sentiment analysis, and they make up the bulk of volume on long-established properties.

## How ScraperAPI compares with dedicated Tripadvisor scrapers

If your only goal is Tripadvisor review text at the lowest price and you'll never scrape anything else, a dedicated actor is cheaper per review. Here's the verified landscape:

| Tool | Model | Verified price | Returns review text | Notes |
| --- | --- | --- | --- | --- |
| ScraperAPI | General scraping API | 1 credit base; ~11 with JS rendering | You parse the HTML | Same key works for Google (25 credits), Amazon (5), LinkedIn (30) |
| Apify reviews actor | Cloud actor | $0.90 / 1K reviews (verified Aug 2026) | Yes, with owner responses | 4.96 rating, 100% run success; DataDome handled internally |
| Bright Data | Managed scraper APIs | ~$0.75–$1 / 1K records | Yes, per documented fields | Free tier, no card; compliance trail |
| Outscraper | Pay-per-service | ~$3–4 / 1K reviews | Yes | No monthly commitment |
| Scrapfly | Scraping API | Free 1,000 credits, from $30/mo | You parse it | 98% success in Scrapeway's Aug 2026 test |

A few honest notes. In the AIMultiple benchmark (500 Tripadvisor review URLs, 2,000 requests, May 2026), Oxylabs' Web Unblocker led with a 91% success rate at 7 seconds per request, ahead of Zyte (86%), Decodo (76%), and Nimble (73%) — ScraperAPI wasn't part of that test, so no claims either way. And if you need pre-filtered, pre-translated review datasets with language filters, star filters, and machine translation built in, the Apify actors do that out of the box; with an API you'd build those steps yourself. Where ScraperAPI earns its keep is when Tripadvisor is one of several targets in your pipeline rather than the only one.

## Limits worth knowing before you pick a plan

Three constraints don't show up in the headline pricing:

- **Geotargeting is gated.** Hobby and Startup can only route through US and EU (region-level, no specific European countries). Country-level targeting across 50+ countries — Germany, Japan, Brazil, and so on — requires Business or higher. The `country_code` parameter itself is free on all plans; what's gated is the IP pool behind it.
- **Concurrency is a hard wall.** Exceed your thread limit and requests come back as 429 immediately. Hobby's 20 concurrent threads are fine for scheduled jobs, tight if you're firing parallel crawls.
- **Bigger plans unlock pay-as-you-go.** Only Scaling and above can keep scraping past their credit limit at a fixed rate with a monthly cap you set. On Hobby through Business, you stop when credits stop (or auto-upgrade if you've opted in).

A third-party pricing breakdown of ScraperAPI also flags that some users report reliability dropping on harder targets at scale, and that the credit multiplier system takes getting used to. Both are fair reasons to validate on your actual target URLs during the trial rather than assuming.

## FAQ

**How much does it cost to scrape Tripadvisor reviews with ScraperAPI?** A base request is 1 credit; with JS rendering, roughly 11 credits per page, plus 10 more if a protection bypass triggers. Confirm the exact figure for your target URLs with the Domain Multiplier or the `/account/urlcost` endpoint before running at scale.

**Can I just use the official TripAdvisor API?** Not for analysis. The Content API hard-caps you at 5 reviews per location, which works for a "guest reviews" display widget on your own site but can't support sentiment tracking or comparisons.

**Do I need my own proxies?** No. Residential and mobile IPs, rotation, CAPTCHA solving, and retries are included in every plan including the free tier.

**Does the free trial need a credit card?** No — 7 days, 5,000 credits, signup only.

## So which setup should you actually pick

If you need review text only, from Tripadvisor only, on a budget: the dedicated Apify reviews actor at $0.90/1K reviews is the cheaper per-unit route, and Bright Data or Outscraper cover managed or pay-as-you-go variations.

If Tripadvisor is one target among several — you'll also want Google SERPs, Amazon prices, or general site scraping — one ScraperAPI plan replaces a stack of single-purpose tools. Hobby at $49/month (or $44.10 on annual) covers roughly 9,000 rendered pages a month; Startup at 1,000,000 credits is the tier where per-credit economics start looking comfortable; Business is the minimum if you need country-level geotargeting. And whatever you choose, set a date cutoff on reviews before you scale — that single parameter matters more to your invoice than which plan you pick.

👉 [Compare the full plan list and current annual pricing on ScraperAPI](https://www.scraperapi.com/pricing/?fp_ref=coupons), run your target URLs through the free trial first, and you'll know your real per-page cost before spending a dollar.
