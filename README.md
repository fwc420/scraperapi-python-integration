# ScraperAPI Python Example: Complete Step-by-Step Tutorial — How to Use requests, BeautifulSoup, Scrapy & the SDK Together? Which Integration Method Is Best? What Does It Actually Cost? (Includes Full Plan Comparison & Working Code)

You've hit a wall. You wrote a perfectly reasonable Python scraper, and now every third request returns a 403. Or you're getting CAPTCHAs. Or the IP ban comes fifteen minutes in. You search around, someone mentions ScraperAPI, and you end up wondering: *how do I actually use this thing in Python?*

That's what this guide is for. We'll walk through every method — the API endpoint via `requests`, the proxy port, the SDK, and even Scrapy integration — with real, copy-pasteable code. We'll also cover what everything costs so you don't get surprised mid-month.

---

**Why Python Developers Keep Running Into This Problem**

Web scraping with Python is easy until it isn't. The `requests` + `BeautifulSoup` combo gets you 80% of the way there. The remaining 20% is where real-world projects go to die: rotating IPs, CAPTCHA farms, JavaScript-rendered pages, geolocation restrictions. Building that infrastructure yourself is a month of work minimum.

ScraperAPI is the shortcut. You send your target URL to their API, and they return the HTML — routed through a pool of 40 million+ residential IPs, with automatic CAPTCHA solving and optional headless Chrome rendering. You stay focused on parsing. They handle the walls.

The question most tutorials skip: there are actually *three different ways* to wire ScraperAPI into your Python code, each suited to different scenarios. Let's go through them all.

---

**What ScraperAPI Actually Does (The 30-Second Version)**

Before the code, a quick mental model. ScraperAPI sits between your script and the target website. Every request you send goes through their infrastructure:

- **Proxy rotation** — Different IP on every request from a 40M+ pool across 50+ countries
- **CAPTCHA solving** — Automatically handled when detected
- **JavaScript rendering** — Optional headless Chrome (the `render=true` flag)
- **Anti-bot bypass** — Cloudflare, DataDome, PerimeterX detection and bypass
- **Retry logic** — Failed requests are retried automatically

The output is plain HTML (or parsed JSON if you use their structured data endpoints). Your Python code treats it like any normal response.

👉 [Start free — 1,000 credits/month, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

**Setting Up: Before You Write Any Code**

You'll need an API key. Sign up at ScraperAPI and grab your key from the dashboard — the free tier gives you 1,000 credits per month, plus a 7-day trial with 5,000 credits to actually test against your real targets before committing to anything.

Then install the dependencies you'll need for the examples below:

bash
pip install requests beautifulsoup4 scraperapi-sdk


If you're using Scrapy:

bash
pip install scrapy scraperapi-sdk


---

**Method 1: The API Endpoint with Python `requests`**

This is the most straightforward approach. You're making a regular GET request to ScraperAPI's endpoint, passing your target URL and API key as query parameters.

**Basic Example — Scraping a Simple Page:**

python
import requests

API_KEY = 'YOUR_API_KEY'
target_url = 'https://books.toscrape.com/'

request_url = f'https://api.scraperapi.com?api_key={API_KEY}&url={target_url}'
response = requests.get(request_url)

print(response.text)


That's it. `response.text` is the HTML of the target page, fetched through ScraperAPI's proxy pool. From here, you pipe it into BeautifulSoup exactly as you normally would.

**With BeautifulSoup for Parsing:**

python
import requests
from bs4 import BeautifulSoup

API_KEY = 'YOUR_API_KEY'
target_url = 'https://books.toscrape.com/'

request_url = f'https://api.scraperapi.com?api_key={API_KEY}&url={target_url}'
response = requests.get(request_url, timeout=70)

soup = BeautifulSoup(response.text, 'html.parser')

# Extract all book titles and prices
books = soup.select('article.product_pod')
for book in books:
    title = book.h3.a['title']
    price = book.select_one('p.price_color').text
    print(f"{title}: {price}")


> **One important note on timeouts**: ScraperAPI recommends setting a 70-second timeout in your requests. Their docs call this out explicitly — especially for hard-to-scrape domains, giving them the full window maximizes success rates. A shorter timeout means you might cancel before they finish.

**With JavaScript Rendering — For React/Vue/Angular Pages:**

If your target page loads content dynamically via JavaScript, add `render=true`:

python
import requests
from bs4 import BeautifulSoup

API_KEY = 'YOUR_API_KEY'
target_url = 'https://www.example-js-heavy-site.com/products'

request_url = f'https://api.scraperapi.com?api_key={API_KEY}&render=true&url={target_url}'
response = requests.get(request_url, timeout=70)

soup = BeautifulSoup(response.text, 'html.parser')
print(soup.find('div', class_='product-list'))


Fair warning: `render=true` costs 10 additional credits per request. On a JavaScript-heavy Amazon page, you're also hitting the 5-credit domain multiplier. More on credit math below.

**With Geotargeting — Getting Region-Specific Results:**

python
import requests

API_KEY = 'YOUR_API_KEY'
target_url = 'https://www.google.com/search?q=python+web+scraping'

params = {
    'api_key': API_KEY,
    'url': target_url,
    'country_code': 'us',  # or 'gb', 'de', 'jp', etc.
}

response = requests.get('https://api.scraperapi.com', params=params, timeout=70)
print(response.text)


> Note: The `params` dictionary approach is cleaner because Python's `requests` library handles URL encoding automatically — handy when your target URL contains its own query parameters.

---

**Method 2: The Proxy Port — Drop-In Replacement for Existing Scrapers**

This method is for developers who already have working scraper code that uses a proxy configuration. You just swap in ScraperAPI's proxy server address and your API key becomes the password. Zero structural changes to the rest of your code.

python
import requests

API_KEY = 'YOUR_API_KEY'

proxies = {
    "http": f"http://scraperapi:{API_KEY}@proxy-server.scraperapi.com:8001",
    "https": f"http://scraperapi:{API_KEY}@proxy-server.scraperapi.com:8001",
}

response = requests.get(
    'https://www.example.com',
    proxies=proxies,
    verify=False  # Required — ScraperAPI handles SSL on their end
)

print(response.text)


The `verify=False` is non-negotiable here. ScraperAPI handles SSL certificates on their side of the connection, so you need to disable client-side verification. Your requests library will throw a warning about this — you can suppress it cleanly with:

python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)


**Proxy Port with Parameters:**

To enable extra features in proxy mode, you embed them in the username field separated by dots:

python
import requests

API_KEY = 'YOUR_API_KEY'

# Enable JS rendering and target US geo
proxies = {
    "http": f"http://scraperapi.render=true.country_code=us:{API_KEY}@proxy-server.scraperapi.com:8001",
}

response = requests.get(
    'https://www.target-site.com/page',
    proxies=proxies,
    verify=False
)

print(response.text)


This method is especially useful when you're integrating ScraperAPI into a Scrapy project using the middleware approach, or if you're maintaining a legacy codebase where refactoring the request structure isn't practical.

---

**Method 3: The Python SDK — Cleanest API for New Projects**

If you're starting a new project, the official `scraperapi-sdk` is the most readable option. Install it:

bash
pip install scraperapi-sdk


**Basic GET Request:**

python
from scraperapi_sdk import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

content = client.get('https://www.example.com')
print(content)


**With Premium Proxies (for harder targets):**

python
from scraperapi_sdk import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

# Use premium residential proxies
content = client.get('https://www.amazon.com/dp/B09G9HD3V1', params={'premium': True})
print(content)


**Using `make_request` for Full Response Object:**

When you need access to status codes, headers, or response metadata:

python
from scraperapi_sdk import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

response = client.make_request(url='https://www.example.com')
print(response.status_code)   # e.g., 200
print(response.headers)
print(response.text)


**With Exception Handling:**

python
from scraperapi_sdk import ScraperAPIClient
from scraperapi_sdk.exceptions import ScraperAPIException

client = ScraperAPIClient('YOUR_API_KEY')

try:
    content = client.get('https://www.example.com')
    print(content)
except ScraperAPIException as e:
    print(f"ScraperAPI error: {e.original_exception}")


---

**Method 4: Scrapy Integration**

If you're running large-scale crawls with Scrapy, the SDK's `scrapyGet` method prepares URLs for Scrapy's request pipeline:

python
import scrapy
import os
from scraperapi_sdk import ScraperAPIClient

client = ScraperAPIClient(os.getenv('SCRAPERAPI_API_KEY'))

class ProductSpider(scrapy.Spider):
    name = 'product_spider'

    async def start(self):
        urls = [
            'https://books.toscrape.com/catalogue/page-1.html',
            'https://books.toscrape.com/catalogue/page-2.html',
        ]
        for url in urls:
            yield scrapy.Request(
                url=client.scrapyGet(url, render=True),
                callback=self.parse
            )

    def parse(self, response):
        for book in response.css('article.product_pod'):
            yield {
                'title': book.css('h3 a::attr(title)').get(),
                'price': book.css('p.price_color::text').get(),
            }


The `scrapyGet` method converts your target URL into a ScraperAPI request URL with all parameters embedded. Your Scrapy spider doesn't need to know anything else about the proxy logic.

---

**Async Scraping — For Bulk Jobs That Don't Need to Wait**

When you have thousands of URLs to process and waiting synchronously for each one is too slow, the async API is your friend. You submit a job and poll for the result:

python
from scraperapi_sdk import ScraperAPIAsyncClient, ScraperAPIException

api_key = 'YOUR_API_KEY'
client = ScraperAPIAsyncClient(api_key)

# Submit the job
try:
    job = client.create('https://www.example.com')
    request_id = job.get('id')
except ScraperAPIException as e:
    print(e.original_exception)
    request_id = None

# Poll until complete
if request_id:
    result = client.wait(
        request_id,
        cooldown=5,       # Check every 5 seconds
        max_retries=20,   # Give up after 100 seconds
    )
    print(result)


This is especially useful when you're submitting batch Amazon product scrapes or Google SERP jobs — the async API handles the queue, you just collect results.

---

**Structured Data Endpoints — Skip the HTML Parsing Entirely**

For popular platforms, ScraperAPI returns pre-parsed JSON so you don't need to write any HTML parsing code at all. The SDK makes this particularly clean:

python
from scraperapi_sdk import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

# Amazon product data — returns structured JSON with 18+ fields
product = client.amazon.product('B09G9HD3V1')
print(product['title'])
print(product['price'])
print(product['rating'])

# Google SERP data
serp_results = client.google.search('python web scraping tutorial')
for result in serp_results['organic_results']:
    print(result['title'], result['link'])

# Walmart product search
walmart_data = client.walmart.search('standing desk')


Available structured endpoints cover: Amazon (product, search, offers, prices), Google (SERP, News, Jobs, Shopping), Walmart (search, product, category), eBay (search, product), and Redfin (listings, agent details).

---

**Understanding the Credit System Before You Build**

This is where most tutorials let you down. The pricing page shows "100,000 credits" for $49/month. What it doesn't explain prominently is that credits aren't consumed 1:1 with requests.

The real cost depends on the target domain and the features you enable:

| **Domain / Parameter** | **Credits per Request** |
| --- | --- |
| Normal page (blog, news, etc.) | 1 |
| E-commerce (Amazon, Walmart, eBay) | 5 |
| SERP (Google, Bing + subdomains) | 25 |
| Social media (LinkedIn) | 30 |
| `render=true` (JavaScript rendering) | +10 |
| `premium=true` (residential proxies) | +10 |
| `ultra_premium=true` | +30 |
| Cloudflare / DataDome / PerimeterX bypass | +10 |
| `premium=true` + `render=true` combined | +25 total |
| `ultra_premium=true` + `render=true` combined | +75 total |

**Practical example**: Scraping Amazon product pages with JavaScript rendering enabled costs `5 (Amazon) + 10 (render=true) = 15 credits` per request. The $49/month Hobby plan's 100,000 credits gets you roughly **6,666 Amazon product pages**, not 100,000. Run your actual target through ScraperAPI's dashboard cost estimator before committing to a plan.

The good news: you only pay for successful requests (HTTP 200 or 404 responses). Failures don't drain your credits.

---

**Full Plan Comparison Table**

All plans include proxy rotation, JavaScript rendering capability, JSON auto-parsing, custom headers, CAPTCHA bypass, session management, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. The annual billing option knocks 10% off every tier automatically at checkout.

| **Plan** | **Monthly Price** | **Annual (per mo)** | **API Credits/mo** | **Concurrent Threads** | **Geotargeting** | **Pay-As-You-Go** | **Get Started** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Free** | $0 | — | 1,000 | 5 | Limited | No | [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49 | $44.10 | 100,000 | 20 | US & EU only | No | [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149 | $134.10 | 1,000,000 | 50 | US & EU only | No | [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299 | $269.10 | 3,000,000 | 100 | Global (50+ countries) | No | [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** | $475 | $427.50 | 5,000,000 | 200 | Global | Yes | [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975 | $877.50 | 10,500,000 | 300 | Global | Yes | [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975 | $1,777.50 | 21,500,000 | 500 | Global | Yes | [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22,000,000+ | 500+ | Global | Yes | [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

A few things worth knowing about this table:

- **Geotargeting is tier-locked**: Hobby and Startup only give you US and EU proxies. Need to target a specific country in Asia or Latin America? That requires Business ($299/month) or higher.
- **Pay-As-You-Go only kicks in at Scaling ($475/month) and above**. On the three lower paid tiers, running out of credits mid-month means a hard stop — you either upgrade or wait for the billing cycle to reset.
- **Credits don't roll over**. Unused credits expire at renewal. Size your plan to your actual average monthly usage, not your theoretical maximum.
- **Analytics history** is capped at 30 days on Hobby/Startup; Business and above get unlimited history.
- The **7-day free trial** gives you 5,000 credits to test against real targets — no credit card needed.

---

**Which Integration Method Should You Use?**

| **Your Situation** | **Best Method** |
| --- | --- |
| New project, clean codebase | SDK (`scraperapi-sdk`) |
| Existing scraper using proxy config | Proxy port method |
| Ad-hoc scripts and quick tests | API endpoint via `requests` |
| Large-scale crawls with Scrapy | SDK `scrapyGet` + Scrapy spider |
| Processing thousands of URLs async | Async API + `client.wait()` |
| Amazon / Google / Walmart data, no parsing | Structured Data Endpoints via SDK |

---

**Which Plan Should You Pick?**

This depends almost entirely on what you're scraping, not how many URLs you have.

**Hobby ($49/month)** works well for personal projects, prototypes, or side projects targeting plain HTML pages. If you're scraping blogs, news sites, or basic e-commerce pages without JavaScript rendering, 100,000 credits goes a long way. The moment Amazon or Google enters your URL list, recalculate.

**Startup ($149/month)** is the right call once you've validated the project and need reliable volume. A million credits with 50 concurrent threads covers most small-to-medium production workloads — you can run 50 parallel requests and process a few hundred thousand standard pages per month without stress.

**Business ($299/month)** is worth it when you need either (a) more than 50 concurrent connections for serious parallel scraping, (b) global geotargeting beyond the US and EU, or (c) the peace of mind of unlimited analytics history for a production system other things depend on.

**Scaling ($475/month)** is where Pay-As-You-Go becomes available, which is a meaningful operational difference. If your usage is spiky — slow weeks and then sudden surges — not having a hard credit cap mid-cycle is genuinely worth the price jump.

👉 [Try ScraperAPI free — 5,000 credits for 7 days, no card needed](https://www.scraperapi.com/?fp_ref=coupons)

---

**Real Performance: What ScraperAPI Does Well and Where It Struggles**

Independent benchmarks (Scrapeway, April 2026) show a fairly clear pattern:

**Strong performance:**
- Amazon product pages — ~98% success rate
- Zillow real estate listings — 100% success rate
- Etsy — ~99% success rate
- Google SERPs — strong success with the structured endpoint
- Walmart — ~93% success rate

**Where things get harder:**
- Instagram, Twitter/X, Booking.com — 0% success rate in independent testing
- Sites requiring login — explicitly outside ScraperAPI's scope (their ToS forbids it)
- Some job sites — success rates around 84–90%

The pattern makes sense: ScraperAPI has built dedicated scraping logic and structured data endpoints for the high-demand platforms like Amazon and Google. For everything else, it's routing through their proxy pool and hoping the anti-bot system doesn't adapt. For Instagram and Twitter, they simply don't work.

If your project is heavily focused on Amazon or e-commerce in general, ScraperAPI is among the most reliable options available. If you need social media data, look elsewhere.

---

**Practical Tips That Actually Matter**

**Set your timeout to 70 seconds, always.** The docs mention this, but it's easy to miss. Too-short timeouts cause you to cancel requests you'll still be charged for (if cancelled after some processing has started), and you miss successful scrapes on slower targets.

python
response = requests.get(request_url, timeout=70)


**Test your actual targets on the free tier first.** Sign up, get the 5,000-credit trial, and run it against the specific URLs you care about. Watch what the credit cost is per request on *your* targets, not a generic example. That's the only way to know which plan actually makes sense before paying.

**Use the `params` dict with `requests` for cleaner code.** When your target URL has its own query parameters, building the request URL as a string can create encoding issues. The `params` dictionary approach lets `requests` handle it:

python
params = {
    'api_key': API_KEY,
    'render': 'true',
    'country_code': 'us',
    'url': 'https://www.google.com/search?q=test&hl=en'  # safely encoded
}
response = requests.get('https://api.scraperapi.com', params=params, timeout=70)


**Check the `sa-credit-cost` response header.** Every ScraperAPI response includes this header, telling you exactly how many credits that specific request consumed. Log this during development so you can build accurate cost projections:

python
response = requests.get(request_url, timeout=70)
print(f"Credits used: {response.headers.get('sa-credit-cost', 'N/A')}")


**Don't use `ultra_premium=true` + `render=true` unless you have a specific reason.** That combination costs 75 credits per request. On the Hobby plan, that's 1,333 total pages for your entire month. Start with standard proxies, test success rates, and only escalate if the target requires it.

---

**Bottom Line**

ScraperAPI earns its reputation as the "drop-in" scraping infrastructure for Python developers. The API endpoint method is genuinely as simple as a two-line change to an existing `requests` call. The SDK is clean enough to use confidently in production. The proxy port method is a lifesaver for migrating legacy code.

Where you need to pay attention is the credit system. The numbers on the pricing page are real, but they describe ideal conditions — plain pages, no rendering, no premium proxies. For anything more complex, the multipliers kick in and your effective capacity shrinks. Calculate your real cost per request on your actual targets, pick the plan that fits that math, and use the 7-day trial to validate before you commit.

The infrastructure is solid. The docs are genuinely good. The free tier is enough to run a real test. That's a better starting point than most tools in this category.

👉 [Start your free ScraperAPI trial — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)
