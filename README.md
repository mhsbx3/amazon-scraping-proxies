# amazon scraping proxy: match the IP model and targeting to your ASIN list, not the sales page

Amazon is not a difficult scrape target because of clever JavaScript. It is difficult because it spends real money deciding whether the client in front of it is a person, and because a large share of the data you actually want is location-dependent. Run a collector out of a datacenter range and you often won't get a clean block page at all. You'll get a product page that renders normally and quotes a price that a buyer three states away never sees, or an offer block that comes back half empty. Silent wrongness is the expensive failure, because a 403 is obvious and wrong data isn't.

Two questions decide whether a proxy is worth paying for on Amazon work. Can it hold a session long enough to pull a product page, its offer list and a slice of reviews before the IP rotates underneath you, and does the exit IP genuinely sit in the market whose price you're recording. Everything else — protocols, dashboards, referral codes — is secondary.

This piece covers what actually breaks on Amazon, what to check in a provider, and how 9Proxy's package structure maps onto realistic scraping workloads, including the parts of its setup that will annoy you.

## What actually breaks when you scrape Amazon

### It's four separate problems wearing one trench coat

Anti-bot systems score an incoming request on more than its IP. On Amazon, four things tend to fail independently:

**IP reputation and network type.** Datacenter ASNs are the first thing scored down. Residential IPs from real ISPs clear that bar, which is why almost every guide tells you the same thing. The nuance is pool reuse: a residential IP that forty other scraping jobs hammered this morning is no longer clean.

**TLS and header fingerprint.** Request order, HTTP/2 settings, and User-Agent consistency matter. A residential IP paired with a stale Chrome 91 User-Agent still gets challenged.

**Rate and concurrency.** Firing 200 parallel requests at one ASIN from a single IP is the fastest way to get a subnet cooled off. Amazon's throttling frequently applies to a range, not just one address.

**Session continuity.** Product pages carry a lot of their useful data below the fold, and reviews sit behind additional requests. If your IP rotates mid-job, the second request arrives with different cookies and a different geo context, and you get a page variant or a challenge page instead of the data.

### The problem nobody mentions until the numbers are wrong

Amazon prices and availability vary by delivery location for a large chunk of its catalog. The price your script records depends on the IP it came from. Pull the same ASIN through an IP in one metro and then another and you can get different prices, different offer counts, different delivery promises, and occasionally different buy-box owners.

So city and ZIP-level targeting isn't a nice-to-have for price intelligence. It's the difference between a dataset that reflects a real market and one that reflects wherever your proxy happened to land. This is the single most common reason a price-monitoring pipeline looks healthy and produces numbers nobody can reproduce manually.

## The proxy checklist for Amazon work

Before comparing prices, narrow the field on capability:

- **Residential, not datacenter**, as the default for product detail pages.
- **Sticky sessions you control**, so one job keeps one exit IP across paginated requests.
- **Geo granularity down to city or ZIP**, and ISP filtering if you're auditing specific carriers.
- **Concurrency limits you can actually hit.** A plan that caps you at a handful of simultaneous ports won't run a nightly job.
- **A bandwidth model that matches your request profile.** Plain HTTP fetches are small; headless browser runs are not. A product page response often runs several hundred kilobytes of HTML before you touch a single asset, and a Playwright or Selenium run pulls CSS, JS and fonts on top of that.
- **HTTP(S) and SOCKS5**, since antidetect browsers and some automation stacks want one or the other.
- **A replacement policy for dead IPs.** Residential addresses go offline for ordinary reasons.

What you can skip: reviews of features nobody uses on Amazon work, and any provider whose pitch is that its proxies solve CAPTCHAs. They don't. A proxy changes who you look like, not whether a challenge page is served.

## Where 9Proxy fits

9Proxy is a residential proxy provider that has been running since 2023 and reports a pool of 20M+ residential IPs across 90+ locations, with targeting by country, state, city, ZIP code and ISP. It supports HTTP(S) and SOCKS5. The part that matters for scraping budgets is that it sells residential access two different ways, and the two are not interchangeable.

### IP-based, with unlimited bandwidth per IP

You buy a fixed number of IPs. An IP is deducted from your balance only when you forward it to a local port, and unused IPs never expire. Once an IP is live it stays online for somewhere between a few hours and roughly 24 hours, because that's how residential addresses behave.

The upside is that bandwidth is unlimited while the IP is active, which suits heavy page loads and long sessions. The catch is stated plainly in 9Proxy's own documentation: this model requires the desktop app on Windows, macOS or Linux, which handles local port forwarding, and you use the proxy at `localhost:port`. If your scraper runs on a headless server or in a container, that's a real architectural constraint, not a footnote.

There are two features that soften it. Auto Refresh replaces an IP that drops, and Auto Rotation rotates proxies on a schedule across selected ports. Both consume more from your balance, which feeds directly into the cost math later. A separate refund policy lets you swap an IP that fails to work within 60 seconds, and the "Today List" lets you reuse IPs that came back online during the day without paying again.

👉 [Check what a bundle of IPs currently costs](https://bit.ly/9-Proxy)

### GB-based, with unlimited endpoints

The bandwidth model is the one most Amazon pipelines should start with. You buy a pool of gigabytes, generate as many endpoints as you like, and choose between rotating mode (new IP per request or session) and sticky mode (same IP until the session timer ends). Traffic is valid for 180 days, or with no expiry at all on the enterprise tiers.

Two things make this the more practical option for server-side scraping. Authentication is username/password or an IP whitelist, so it works from a VPS or a Lambda-style job with no desktop app involved. And because the unit is traffic rather than IPs, a job that runs twice a day costs the same as one that runs twice a week.

The trade-off is obvious enough: every retry, every challenge page, and every image your headless browser drags in counts against the balance.

## 9Proxy pricing, package by package

Prices below are in USD and reflect published rates after the adjustment 9Proxy announced on 18 May 2026 and applied from 1 June 2026. That change raised IP-based and bundle prices; GB-based pricing stayed where it was. Mid-range IP tiers scale down in per-IP rate as volume grows, so treat the table as an entry guide and confirm the current figure before you buy.

### Residential proxy by IPs (unlimited bandwidth per IP)

| Package | Price | Effective rate | Buy |
| --- | --- | --- | --- |
| 100 IPs | $24 | $0.24 per IP | [Get 100 IPs](https://bit.ly/9-Proxy) |
| 500 IPs | $72 | $0.14 per IP | [Get 500 IPs](https://bit.ly/9-Proxy) |
| 1,000 IPs + 500 bonus | $126 | ~$0.08 per IP | [Get 1,500 IPs](https://bit.ly/9-Proxy) |
| 100,000 IPs | $2,300 | $0.023 per IP | [Get 100,000 IPs](https://bit.ly/9-Proxy) |
| 500,000 IPs | $8,625 | ~$0.017 per IP | [Get 500,000 IPs](https://bit.ly/9-Proxy) |

Between 1,500 and 100,000 IPs there are additional stair-stepped packages at 2,500, 5,000, 15,000, 25,000 and 50,000 IPs, each with a lower per-IP rate than the tier below it.

### Residential proxy by GB

| Package | Price | Effective rate | Validity | Buy |
| --- | --- | --- | --- | --- |
| 5 GB | $15 | $3.00 per GB | 180 days | [Get 5 GB](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | $105 | $2.10 per GB | 180 days | [Get 55 GB](https://bit.ly/9-Proxy) |
| 100 GB | $150 | $1.50 per GB | 180 days | [Get 100 GB](https://bit.ly/9-Proxy) |
| 200 GB | $200 | $1.00 per GB | 180 days | [Get 200 GB](https://bit.ly/9-Proxy) |
| 1,000 GB | $800 | $0.80 per GB | 180 days | [Get 1,000 GB](https://bit.ly/9-Proxy) |
| 2,000 GB | $1,500 | $0.75 per GB | 180 days | [Get 2,000 GB](https://bit.ly/9-Proxy) |
| 3,000 GB (Enterprise) | $2,160 | $0.72 per GB | No expiry | [Get 3,000 GB](https://bit.ly/9-Proxy) |
| 6,000 GB (Enterprise) | $4,200 | $0.70 per GB | No expiry | [Get 6,000 GB](https://bit.ly/9-Proxy) |
| 10,000 GB (Enterprise) | $6,800 | $0.68 per GB | No expiry | [Get 10,000 GB](https://bit.ly/9-Proxy) |

### Bundle packages (IPs plus traffic)

| Bundle | Contents | Price | Buy |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 | [Get the Pro bundle](https://bit.ly/9-Proxy) |

Bundled traffic carries the same 180-day validity as standalone GB packages. Payment options include cards, cryptocurrency, local payment methods, Alipay, Google Pay and Apple Pay; some payment routes occasionally carry an extra 5% discount or bonus, which is worth checking at checkout. A free account is available, but trial credit isn't a standing product. Reviewers note that trial codes have historically been promotion-dependent and handed out through support.

## Which plan fits an actual Amazon workload

Estimates below assume a plain HTTP fetch of a product page at roughly 400 KB, and note where retries and browser rendering push that up.

**One seller monitoring a few hundred ASINs.** 500 ASINs daily at 400 KB is about 200 MB a day, or 6 GB a month. The 55 GB package at $105 covers roughly nine months of that traffic, though the 180-day validity means you'd realistically use around 36 GB inside the window. The 100-IP package at $24 is the alternative if you run it from a workstation through the desktop app, where 5 forwarded IPs a day would use a 100-IP balance in about three weeks.

**A headless browser pipeline.** Playwright or Selenium fetching the same 500 pages can easily consume 1.5 MB per page once JS and CSS are counted, which turns 6 GB a month into roughly 22 GB. The 200 GB package at $200 is the comfortable landing spot; budget for roughly 10–20% overhead in retries and challenge pages.

**An agency tracking 20,000 ASINs daily.** Plain HTTP at 400 KB puts you near 240 GB a month. The 2,000 GB tier at $1,500 covers seven months of that inside the validity window. Spreading requests across a large IP pool and paying per IP instead removes the bandwidth ceiling entirely, but you'll be managing the desktop app and IP churn to do it.

👉 [Start with the smallest US-heavy package that covers your list](https://bit.ly/9-Proxy)

> Amazon's Conditions of Use prohibit data mining and scraping without permission, and its robots.txt disallows much of the catalog. If the data feeds your own seller account or an associates catalog, the official routes are SP-API and the Product Advertising API. A proxy changes who gets blocked, not your legal position, so keep request rates reasonable and don't send 200 parallel requests at one ASIN.

## Setting up 9Proxy for an Amazon scraper

1. Create an account and pick a model. GB-based for server-side jobs, IP-based if you're driving an antidetect browser from a desktop.
2. For the IP model, install the desktop app on Windows, macOS or Linux and open the port settings under More → Settings → Port Numbers to define your starting port and count.
3. Filter the pool by US state and city, and use the ZIP code field where you need a specific delivery market. This is the step that decides whether your price data is defensible.
4. Forward an IP to a port and point your stack at `localhost:port`. Add proxy authentication if you don't want an open local port.
5. For the GB model, skip the app entirely. Grab the endpoint from the dashboard and use username/password or whitelist your server IP. Set sticky sessions for multi-request jobs and rotating for one-off lookups.
6. Build retry logic that distinguishes throttling from a dead IP, and log the exit IP alongside every price you store. When a number looks wrong six weeks later, that log is the only way to figure out why.

## What 9Proxy won't do for you

Proxies are one layer of a scraping stack, and it's worth being clear about where the limits sit.

Independent comparisons place budget residential providers in a specific performance band. One 2026 comparison of budget providers puts 9Proxy's residential bandwidth at roughly $1.30–$2 per GB, with Tier 1 success rates above 95% and moderately protected targets in the 85–92% range. The same comparison explicitly names Amazon as a target where budget providers struggle, arguing that heavily protected sites need larger, fresher pools or dedicated unlocking tools. Treat that as one analyst's view with its own affiliate incentives, but the underlying point is fair: if you need 97%+ success on Amazon at high volume, a budget pool is not the tool. One reviewer testing bundle plans reported around 99.5% success and 0.6 seconds average response time, which is a rosier figure from a smaller sample.

Other limits worth knowing before you pay:

- No built-in CAPTCHA solving or unlocking API. You handle challenges in your own code.
- The mandatory desktop app for IP-based plans rules out a pure server deployment.
- IP lifetime is variable by nature. A residential IP can drop in a few hours, which is why the 60-second replacement policy exists.
- Trial access depends on promotions. Don't plan a project around a free tier that may not be offered.
- City-level depth is thinner than what enterprise providers offer, even though ZIP filtering is available. If your work depends on pinning dozens of specific metros, test before you commit.

Alongside that, 9Proxy's pricing is genuinely at the low end for legitimate residential traffic, support runs 24/7 across live chat, email and Telegram, and unused IP balances don't expire.

## FAQ

**Do I need residential proxies to scrape Amazon?**
For product pages, yes in practice. Datacenter ranges get scored down quickly and you'll spend more on engineering time than you save on bandwidth.

**Is 9Proxy good for Amazon scraping?**
It's a reasonable fit at small to mid volume, particularly on GB-based plans where you get server-friendly authentication and flat traffic costs. If your workload is large and needs very high success rates on heavily protected pages, test it against your own targets before committing a budget, and keep a second provider in the mix.

**How many IPs do I actually need?**
Work backwards from concurrency. If each worker holds one sticky IP and you run 20 workers, you need at least 20 live IPs with spare capacity for refreshes. The IP-based model deducts an IP per forward, so a 100-IP balance is roughly 100 forwarding events, not 100 permanent addresses.

**Does 9Proxy have a free trial?**
There's a free sign-up, and trial credit has been offered through promotions and support requests rather than as a standing free tier.

**Will it work with Scrapy, Selenium, Playwright and antidetect browsers?**
Yes. HTTP(S) and SOCKS5 are both supported, and the provider is commonly paired with mainstream automation frameworks and antidetect browsers for e-commerce work.

## Bottom line

For Amazon, the proxy decision is really two decisions: which model your infrastructure can support, and how precisely you can target the market you're measuring. Server-side pipelines should default to GB-based packages with sticky sessions and US city targeting, since that combination is where the accuracy comes from. Desktop-driven account and browser work is where the IP-based model earns its unlimited bandwidth.

9Proxy's entry pricing starts low enough that testing costs less than a lunch, which is the correct way to decide. Run 50 ASINs through a US city target, compare the prices against what you see manually, and let the match rate decide. If the numbers line up, scale the package. If they don't, you've lost $15 and learned something specific about your target instead of guessing.

👉 [Create a 9Proxy account and test US targeting on your own ASIN list](https://bit.ly/9-Proxy)
