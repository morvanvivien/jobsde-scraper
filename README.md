# How to Scrape Jobs.de: Germany Job Postings API, No Login

> **Run it now, no code: [jobsde-jobs-scraper on the Apify Store](https://apify.com/bibian/jobsde-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 job postings &nbsp;|&nbsp; no login, no browser required

Jobs.de is a German job board with over 70,000 live postings and no public export or official API for third parties. This guide shows how to get its postings as clean structured data: title, company, city, salary when disclosed, employment type and remote option, with a free Apify Actor, from the console or from code.

Searches this page answers: jobs.de scraper, Germany jobs API, German job board scraper, jobs.de data extraction.

## What you get from the Jobs.de scraper

- Every job posting, with title, company, location (city, region, postal code), employment type and remote policy.
- Salary range (`salaryMin`, `salaryMax`) when the posting discloses one.
- Industry tags, skills, publication and expiry dates.
- Optional full description as plain text, with any recruiter's named signature, e-mail or direct phone number removed.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, no personal data) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Stable numeric id (use it to deduplicate) and canonical posting URL on jobs.de |
| `title` | Job title as published |
| `companyName` | Hiring company; `null` when the posting hides it (then `confidentialCompany: true`) |
| `companyUrl` | The employer's own page on jobs.de, when the posting links one |
| `locationLabel`, `city`, `region`, `postalCode`, `country` | Work location of the first listed site; `region` is the German state (Bundesland) |
| `additionalLocations` | Any further work locations beyond the first, as `{ city, postalCode, country }` |
| `remote` | `remote`, `hybrid`, `onsite` or `unknown` |
| `employmentType`, `employmentTypeLabels` | Normalized type (`full_time`, `part_time`, `apprenticeship`, `internship`, `temporary`, `freelance`, `unknown`) and the site's own raw German term(s) |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range, only when the posting discloses a structured one |
| `datePosted`, `validThrough` | ISO 8601 publication and expiry dates |
| `experienceLevel` | Years-of-experience tag, when set (rare) |
| `industry` | Comma-separated occupational-field tags set by the site |
| `skills` | Skill tags listed by the posting; empty on most postings, which list none |
| `source`, `sourceId` | Always `jobs.de`, and the native numeric id on the source site |
| `scrapedAt` | Extraction timestamp, ISO 8601 |
| `description` | Full job description as plain text, only when "Include full description" is on |

## Use cases

- **Monitor the German job market.** New postings by job title, across every industry and region.
- **Feed a job alert or an aggregator.** Run it daily and get the newest postings, deduplicated by their stable numeric `id`.
- **Find companies that are hiring.** Which employers recruit for a given role right now, with city, region and salary when published.
- **Build a dataset for analysis.** Employment type, remote policy and industry come as clean, structured fields instead of free text.

## Input

| Field | What it does |
|---|---|
| `keyword` | Job title or skill, for example `entwickler` (developer) or `pflege` (care). Leave empty to use the default keyword, which always returns results. |
| `maxItems` | The run stops once this many postings are saved (default 50). |
| `includeDescriptions` | Fetch and include each posting's full text description. Turn off for a faster, lighter run when you only need the structured fields (default on). |
| `onlyNew` | For scheduled runs. Skip the postings an earlier run of this same search already delivered, so each run returns, and charges, only what is new (default off). |

Example input:

```json
{
  "keyword": "entwickler",
  "maxItems": 50
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/jobsde-jobs-scraper) and click **Try for free** (a free Apify account is enough). The form is already filled with keyword `entwickler`.
2. Fill in the keyword and the maximum number of results, then click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor, tick **Only new results**, and optionally add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/jobsde-jobs-scraper').call({
    keyword: 'entwickler',
    maxItems: 50,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.city} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryCurrency ?? ''}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/jobsde-jobs-scraper").call(run_input={
    "keyword": "entwickler",
    "maxItems": 50,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('city')} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryCurrency') or ''}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$2.00 per 1,000 job postings** (that is $0.002 per job posting), with no monthly subscription. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/jobsde-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- jobs.de answers plain, datacenter-proxied and Chrome-fingerprinted requests alike with a small "Bot Check" interstitial page; the Actor detects this and automatically retries through a residential IP, so results come back even though the cheaper paths never work on this particular site.
- Salary, `companyUrl`, `skills` and `experienceLevel` are each published on only a minority of postings; the matching fields are then `null`/`[]` rather than guessed (visible in `sample-output.json`).
- The dataset contains job postings only. Any line in the description that carries a named recruiter's signature, e-mail or direct phone number is removed before it is saved; the hiring company's own name and address are kept.
- This is an unofficial tool, not affiliated with jobs.de. Check the site terms and the data protection rules that apply to your use, for example the GDPR.

## FAQ

**Does it need a browser, cookies or a proxy on my side?**  
No. The Actor reads publicly visible pages only and handles jobs.de's own anti-bot check automatically.

**How do I only get new postings every day?**  
Tick `onlyNew` and schedule the Actor once a day. Each run then returns, and charges, only the postings that earlier runs of the same search did not already deliver.

**Which formats can I export?**  
JSON, CSV, Excel, XML, HTML table or RSS, from the dataset page or through the API.

**Is it legal to scrape?**  
The Actor collects publicly visible postings and respects the site's robots.txt rules. How you use the data is your responsibility: check the site's terms and the laws that apply to you, for example the GDPR.

## Try it

**[Open jobsde-jobs-scraper on the Apify Store](https://apify.com/bibian/jobsde-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
